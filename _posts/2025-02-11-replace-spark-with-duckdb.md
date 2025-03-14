---
layout: post
title: Spark is dead, long live DuckDB!
---

I was attending a DuckDB meetup in Leuven, where a presenter was detailing how they replaced Spark with DuckDB to cut costs, when rather abruptly, someone in the audience asked, “How do I know if I should really use DuckDB now, me being not a data engineer? What if my query is heavier than expected? What if my data fluctuates and just sometimes goes beyond a single machine?”. The presenter admitted that you set up the architecture once, do your best benchmarking, and hope for the data dynamics to stay the same.

I was intrigued, but not really convinced, yet.

## Catalog is the bridge

Next day after the event, I dove into the idea that the query engine should be automatically selected based on the query itself. Having some scars from hefty BigQuery bills, I built a SQL executor that switched between DuckDB and BigQuery. The challenge though was that I couldn’t use the same query with both engines due to the **lack of shared catalog** support.

Fast forward a year, and we now have **Unity Catalog support for DuckDB and Polars** — yes, the same catalog well supported by distributed engines like Spark and platforms like Databricks.

I quickly threw together a prototype — let's see it in action:

1) Launch the Unity catalog:

```bash
$ git clone https://github.com/unitycatalog/unitycatalog.git unity-catalog 
$ cd unity-catalog
$ bin/start-uc-server
```

should start it at `:8080`.

2) Load a decent chunk [Clickbench](https://benchmark.clickhouse.com/) data into UC:

```python
spark.read.parquet("data/hits.parquet") \
    .limit(5_000_000) \
    .write \
    .format("delta") \
    .option("overwriteSchema", "true") \
    .mode("overwrite") \
    .option('path', '<abs_path>/hits_delta') \
    .saveAsTable("default.hits")
```

that’s all it takes to expose a Unity Catalog table backed by a Delta table (files) to other query engines!

3) Pull-in [frugal-engine](https://github.com/astronautas/frugal-engine):

```bash
$ pip install git+https://github.com/astronautas/frugal-engine
```

4) Set-up the executor:

```python
import logging
from frugal_engine.main import setup

logging.basicConfig(level=logging.INFO)
setup()
```

This sets up the hybrid engine with a single-node executor and a cluster executor as a fallback.

Now, let's run a few experiments 🧪. We'll try a simple select first:

```python
# %%sequel deliberately, not to interfere with %%sql
%%sequel 
SELECT * 
FROM unity.default.hits LIMIT 10
```

You should see in the logs that it picked a single node executor since the data size is relatively small:

```bash
INFO:frugal_engine.main:single_node_executor has been chosen, because the top source count is less than or equal to the threshold: 6000000
INFO:frugal_engine.main:Executing query in single_node_executor: SELECT * 
FROM unity.default.hits LIMIT 10
```

Let’s assume our machine is tiny and lower the threshold for single-node workflows:

```python
from frugal_engine.main import setup
setup(threshold=100_000)

%%sequel 
SELECT * 
FROM unity.default.hits LIMIT 10

INFO:frugal_engine.main:Executing query in cluster_executor: SELECT * 
FROM unity.default.hits LIMIT 10
```

From the logs, we see it falls back to a cluster engine since it "thinks" a single machine won’t process that many rows in a reasonable time. It should also fall back if the single-node executor reaches its memory limits—or even proactively, using ML to forecast that it won’t finish in a reasonable time.

Identical SQL, totally different engines! Convenient-ish to use - check. What about the promise of improved performance / reduced costs?

## Quantifying the cost savings

We can’t draw firm conclusions on costs yet, as proper testing with a Spark cluster, Delta access from S3, and hybrid scenarios is needed—it's a complex evaluation.

That said, I ran some [local benchmarks](https://github.com/ClickHouse/ClickBench/blob/main/spark/queries.sql) to gauge query time savings. For 5 million rows, the hybrid engine took ~10 a forcefully cluster executor took ~52 seconds.

With usability and (some) cost aspects explored, let's examine what makes the package tick.

## DuckDB UC - not quite there yet
I initially tried to use DuckDB for the single-node executor, but... it didn’t go well.

I was so hyped by [many](https://www.youtube.com/watch?v=oxPFt_HEFAc) [talks](https://www.youtube.com/watch?v=oxPFt_HEFAc) about how DuckDB should at least have read-only support for Unity Catalog. It works fine for pre-loaded OSS UC tables:

```python
import duckdb
import logging

duckdb.sql("""
	install uc_catalog from core_nightly;
	load uc_catalog;
	install delta;
	load delta;
""")

duckdb.sql(f"""
CREATE SECRET (
	TYPE UC,
	TOKEN 'not-used',
	ENDPOINT 'http://127.0.0.1:8080',
	AWS_REGION 'us-east-2'
);
""")

duckdb.sql("ATTACH 'unity' AS unity (TYPE UC_CATALOG);")


logging.basicConfig(level=logging.DEBUG)
duckdb.sql(f"select * from unity.default.marksheet limit 10;")
```

But as soon as you commit something to UC via another engine, like Spark, DuckDB halts:

```python
IOException Traceback (most recent call last) Cell In[4], line 4 1 import logging 3 logging.basicConfig(level=logging.DEBUG) ----> 4 duckdb.sql(f"select * from unity.default.hits_2 limit 10;") 

IOException: IO Error: Invalid field found while parsing field: type_precision

CatalogException Traceback (most recent call last) Cell In[5], line 4 1 import logging 3 logging.basicConfig(level=logging.DEBUG) ----> 4 duckdb.sql(f"select * from unity.default.hits_2 limit 10;") 

CatalogException: Catalog Error: Table with name hits_2 does not exist! Did you mean "information_schema.views"?
```

I spent hours trying to figure it out. Fair enough, the extension is tagged as experimental, and there’s a related [GitHub issue](https://github.com/duckdb/uc_catalog/issues/8), but I expected clearer error messages. 👎

## Polars to the rescue

Polars, on the other hand, works flawlessly! Unlike DuckDB, you can’t just attach a catalog—there’s no dedicated Polars client or anything like `polars.sql` yet. To achieve this, we’ll create a wrapper around Polars UC:

```python
class PolarsClient:
    """
    Just an interface similar to Spark and DuckDB i.e. exposing sql, but going through UC catalog
    """
    def __init__(self, unity_catalog_uri: str, memory_limit: int = None):
        self.unity_catalog_uri = unity_catalog_uri
        self.catalog = pl.Catalog(unity_catalog_uri, require_https=False) # last flag - for local testing
        self.memory_limit = None

    def sql(self, query: str) -> pl.DataFrame:
        tables = [table for table in sqlglot.parse_one(query).find_all(sqlglot.exp.Table)]

        assert len(tables) == 1, "Only one table is supported as a source for now"

        df_scanned = [self.catalog.scan_table(str(table.catalog), str(table.db), str(table.this)) for table in tables]
        df_scanned = df_scanned[0]

        var_name = next(name for name, val in locals().items() if val is df_scanned)

        query_for_pl = sqlglot.parse_one(query).from_(var_name).sql()
        return pl.sql(query_for_pl).collect()
```

See what we did here? We used `sqlglot` to retrieve the source tables, pre-load them into `LazyFrame`, and replace references in the original SQL query. Nasty, but it gets the job done!

```python
def setup(unity_catalog_uri: str = "http://localhost:8080", threshold: int = 6_000_000):
    spark = _build_local_spark_client(unity_catalog_uri)
    polars_client = PolarsClient(unity_catalog_uri)
    client = HybridClient(spark_client=spark, polars_client=polars_client, threshold=threshold)

    # non-invasive way to add a magic function
    get_ipython().register_magic_function(lambda line, cell=None: client.sql(line or cell),
        magic_kind="line_cell", magic_name="sequel")
```

`setup` isn’t that interesting—it just introduces a new magic function, `sequel`, that delegates queries to `HybridClient`. The real interesting part is engine the decision function.

## Using query information to select the best engine

```python
    def _count_heuristic(self, query: str) -> Choice:
        tables = [table for table in parse_one(query).find_all(exp.Table)]
        counts = [self.polars_client.sql(f"select count(*) from {table.catalog}.{table.db}.{table.name}").to_series()[0] for table in tables]
        top_count = max(counts)

        logger.debug(f"Top source count: {top_count}")
        
        if top_count <= self.threshold:
            logger.info(f"{Choice.POLARS.value} has been chosen, because the top source count is less than or equal to the threshold: {self.threshold}")
            return Choice.POLARS
        else:
            logger.info(f"{Choice.SPARK.value} has been chosen, because the top source count is greater than the threshold: {self.threshold}")
            return Choice.SPARK
```

Essentially, we select the most appropriate engine for the query based on source table statistic - the row count - and then delegate. It's a simple heuristic, but it illustrates the point. 

```python
    choice = self._count_heuristic(query)
    
    if choice == Choice.POLARS:
        try:
            logger.info(f"Executing query in {Choice.POLARS.value}: {query}")
            # polars - unsupported dialect, duckdb might be closest?
            query = sqlglot.transpile(sql=query, write="duckdb")[0]
            return self.polars_client.sql(query).to_pandas()
        # fixme - too generic exception
        except Exception as e:
            logger.error(f"Error: {e}, fallbacking to {Choice.SPARK.value}")                
            query = sqlglot.transpile(sql=query, write="spark")[0]

        return self.spark_client.sql(query).toPandas()
    else:
        logger.info(f"Executing query in {Choice.SPARK.value}: {query}")
        return self.spark_client.sql(query).toPandas()
```

We use `sqlglot` to transpile between dialects, but unfortunately Polars SQL is not a supported dialect yet. Finally, if you look closely, we fallback to the default distributed Spark executor if we encouter an unforseen issue - e.g. unsupported SQL operation, memory issues, etc.

## Next Step – Databricks

Hope you at least got intrigued by the idea that it's possible to balance the benefits of single and  cluster type query engines without changing analyst workflows. Next, when I have time, I'll extend this to Databricks and test cost savings in a more realistic production setting - think Spark cluster, Delta tables in S3, annoying authentication issues, etc.

Stay tuned!