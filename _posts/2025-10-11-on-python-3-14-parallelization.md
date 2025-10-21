---
layout: post
title: Python 3.14 will change the way you parallelise code
---

Free-threading in Python is finally here. Here’s why it matters and how I plan to use it.

<!--more-->

Free threading (no-GIL) is the main feature of [Python 3.14](https://www.python.org/downloads/release/python-3140/), setting it apart from its predecessors. We can process multiple tasks at the same time (so, parallelise) in Python with pure threads. [Just see the benchmarks](https://blog.miguelgrinberg.com/post/python-3-14-is-here-how-fast-is-it) and [here](https://valatka.dev/2024/12/28/async-io-is-not-enough.html).

Not everyone might share my enthusiasm - but here's why it matters:

**"I can just use multiprocessing.”** Yes, but it comes at a cost — slow inter-process communication. Every argument must be pickled for message passing; objects can be slow to pickle, and some aren’t picklable at all. You need to minimize sync points and be selective about the data exchanged between processes.

**"We already have C and Rust extensions.”** Like C NumPy, obviously use one if you can. But would you ever write one yourself just to parallelise?

**"We have async I/O.”** Useful only if you're I/O bound i.e. tons of external calls - [I’ve written about that before](https://valatka.dev/2024/12/28/async-io-is-not-enough.html).

**"I can scale horizontally”**. Sure — but if you care about minimizing single-request latency, horizontal scaling only increases throughput. The time to process an individual request remains the same as long as the machine itself is unchanged, we merely add additional machines to serve more requests.

So, in theory, we have absolutely `None` ergonomic means for parallelisation in Python. 1Starting with 3.14, no-GIL aims to change that.

Here are a few practical use cases where I’ll immediately start exploring no-GIL:

**Algorithm or ML model inference**. You want fast, customer-facing predictions. Batch APIs are full of Python glue for feature fetching and pre/post-processing, but inputs are easy to parallelise. It's all CPU-cycles. At my previous job, we used multiprocessing to cut recommendation endpoint latency by several times (cost = super selective about message passing) — wish we had no-GIL back then.

**Existing async I/O apps**. I suspect many async I/O apps also do decent amount CPU work that can’t currently be parallelised due to the GIL. I’m already excited to benchmark existing implementations — with [parallel event loops on the way](https://labs.quansight.org/blog/scaling-asyncio-on-free-threaded-python), we might see latencies drop!

I don't think **data pipelines** will benefit from no-GIL though. Most don't have latency requirements (rather throughput) and we already offload bulk of CPU work to specialised SQL / DF engines as Polars, DuckDB, PySpark (you are not going to be rewriting them soon).

Hopefully, this convinces you to try free threading / no-GIL. I am excited myself. Especially with `uv`, it's as a easy as:

```bash
uv run --python=3.14t myscript.py
```

Happy 🧵-ing!