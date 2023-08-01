Recently a data platform vendor introduced us to their latest offering: a feature store module for machine learning projects. They highlighted the usual selling points, like reducing train-serve skew, feature backfilling, and enhancing feature documentation. However, one of our experienced senior data engineers remained unconvinced. The question on their mind was straightforward: "Is a feature store truly necessary when we already have a fully operational data platform?". Indeed, there might be a case that most of the aforementioned feature store offerings can be solved with a modern data platform, reducing duplicate efforts and costs.

Let's explore the typical offerings of feature stores and see which ones can be substituted with a familiar data warehouse tables and some additional data tooling around them.

* Single train-serve pipeline
  * Table
  * Feature store
* Backfilling
  * Table
  * Feature store
* Feature documentation
  * Table
  * Feature store
* Real-time access
  * Feature store

I will argue that a data warehouse (DWH) table should be sufficient for most cases, except when real-time feature access is required, which can help save costs when considering off-the-shelf solutions or time when exploring open-source options like [Feast](https://github.com/feast-dev/feast).

### Single train-serve pipeline

A standard argument - you don't want duplicate train-serving pipelines, since this might introduce skew, which reduces your model's accuracy. A table partitioned by period e.g. a day would work just fine:

```sql
# Training
SELECT *
FROM daily_brand_clicks

# Inference
SELECT *
FROM daily_brand_clicks
WHERE date = MAX(SELECT date FROM clicks) and user_id = 123 and brand_id = 456
```

same dataset, same SQL transformation, shave off the latest data point (day) for inference. For training, you will get data points for all existing days and keys. No need for a feature store here.

###  Backfilling

Backfilling involves recomputing feature values for all available time slices in the past, not just for newly arrived data, in datasets with time dimensions. This process becomes crucial during early experimentation phases, allowing for feature rewriting to enhance accuracy and fix bugs. Many feature stores provide this essential functionality.

You can easily support backfilling with e.g. DBT as a data transformation tool - since the term isn't unique to feature stores, it comes from the data engineering world:

1. Always recompute, which proves effective for small tables, or
2. Define an incremental table that allows for a full refresh when needed.

### Feature documentation

To be fair, a typical warehouse table documentation tooling tackles this well. [DBT docs](https://docs.getdbt.com/reference/commands/cmd-docs) allow to serve web-based docs for tables and table columns and much more. Registering an ML feature is as simple as adding a new DBT model.

### Real-time access

The DWH table as a feature store might not work for real-time access. Most data warehouses have analytical databases at their core. At inference, you mostly perform feature lookups for entities by keys, which either relational databases (via B-tree / hash indexes) or NoSQL key-value stores (hash indexes) are optimized for. Analytical databases may not perform as well, as they need to retrieve rows composed of keys and values that are not collocated together on disk or in memory.

At my current place, [Otrium](https://www.otrium.com/men/home), we opted for managing ML features within our data jobs, since either way all ML models run as batch jobs. At [Vinted](https://www.vinted.fr/) a couple of years, we had to use a feature store for recommendation systems to meet high lookup demands in real time. Using Redis enabled efficient querying, effectively serving as a real-time access layer for data warehouse values, though pipelines still resided within our data jobs - so had it not been for the real-time access requirements, we would have just added a lightweight API on the data platform for ML services to consume features.

### Parting words

The decision to adopt a feature store should be carefully evaluated, considering the specific needs and scale of your machine learning projects. A data warehouse table can suffice for many cases, while a feature store might be essential mostly only for scenarios requiring real-time feature access.