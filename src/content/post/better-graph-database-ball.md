---
title: "Better Graph Database Ball"
description: "LadybugDB optimizations targeting the performance bugs found in this research"
pubDate: "June 19 2026"
categories: ["benchmarks"]
heroImage: "/img/2026-06-18-baseball/graph-baseball.png"
authors: ["team"]
tags: ["benchmarks"]
---
Recently Johh Nevin at The Consensus [published](https://theconsensus.dev/p/2026/05/29/ladybug-duckdb-and-postgresql.html) a research note comparing upcoming Graph Query capabilities in Postgres against DuckDB/DuckPGQ and LadybugDB.

The short summary is that DuckDB claimed first place in every benchmark.

At LadybugDB, we are very inspired by the advances DuckDB folks keep making and we try to catch up and innovate.

## Background

LadybugDB uses a variant of strongly typed Cypher. DuckDB and Postgres implement PGQ, which is a read-only graph query language. They maintain their SQL optimized storage as-is and compute some in-memory data structures on the fly.

So in theory LadybugDB should be faster by materializing all those graph data structures on-disk right? But they also have years, sometimes decades of lead in terms of query optimization. DuckDB's performance leadership shows that columnar storage, vectorization and morsel driven parallelism are important.

Those are areas where LadybugDB is doing pretty well. Where we needed to close the gap was indexing, cost based query optimization and some join optimizations. Details below.


## Reproduce the Workload

While there was a textual description of the workload, we couldn't find a ready to run version of the benchmark. So we created [one](https://github.com/adsharma/baseball-bench). 
## Space Amplification

LadybugDB uses Hash Indexing by default. This index format was created to optimize for the ASP-Join, a Worse Case Optimal Join (WCOJ) algorithm described in Kuzu's VLDB paper. That join algorithm is aimed at deep graph traversals/joins that this workload explores.

So why didn't LadybugDB do better?

Hash Indexes in general are a poor fit for the typical use case of a integer primary key. They lead to space amplification, undo some of the benefits of the columnar compression and do not help with range queries.

If your data is sorted by primary key in columnar storage, not every workload needs a primary key index! So we implemented an option to disable the (default) hash index as primary key.

Further, LadybugDB stores edges in both forward and backward directions. If the queries don't care about bi-directional edges, it ends up being a net loss (similar to creating indexes that are not queried).

| System / Mode                                      | Size   |
|----------------------------------------------------|--------|
| DuckDB, no PKs                                     | 618M   |
| DuckDB, with PKs                                   | 702M   |
| Ladybug baseline                                   | 1.3G   |
| Ladybug hash off                                   | 1.1G   |
| Ladybug hash off + fwd rel storage                 | 744M   |
| Ladybug hash off + fwd rel storage + ART indexes   | 838M   |

The rest of the doc is about benchmarking the last config with ART indexes and manageable space amplification.

## Cost Based Query Optimizer

We looked through the performance of some of the queries and implemented a few optimizations. When you insert rows in LadybugDB, we maintain a HyperLogLog based cardinality estimator that's auto updated.

We implemented an `ANALYZE` command that fetches these stats and helps [reorder filters](https://github.com/LadybugDB/ladybug/pull/577) so the most selective filter is applied first.

Then we looked through the RE24 query. LadybugDB graph traversal machinery is optimized for the more general case of graph traversal across heterogenous nodes with differing fanout.

RE24 on the other hand implements a cycle, which works like traversing a long homogenous linked list. We [implemented](https://github.com/LadybugDB/ladybug/pull/574) a more memory efficient path. 


| Query                | Ladybug         | DuckDB               |
|----------------------|-----------------|----------------------|
| Platoon advantage    | 3.417s ± 0.042s | 0.260s ± 0.087s      | 
| Stranded runners     | 1.490s ± 0.026s | 0.057s ± 0.029s      |
| RE24                 | 5.623 ± 0.189s  | 1.357s ± 0.116s      |

This narrows the gap quite a bit. We hope to improve on these numbers in the next couple of release cycles.

## Factorized Joins

In the Kuzu VLDB paper, factorized joins were presented as a big reason why Kuzu was beating DuckDB and DuckPGQ on some of the LDBC benchmarks on deep traversal queries. Since then Prashant Rao at [Lance](https://www.linkedin.com/posts/lancedb_lance-activity-7421045084889964544-85jy) published a [benchmark](https://github.com/prrao87/graph-benchmark) showing that the gains of factorized joins are not as significant as previously thought.

On the other hand, Amine Mhedhbi and his team at PolyMTL published a [SIGMOD 2026 paper](https://amine.io/papers/2026-sigmod-ffx.pdf) which argued that LadybugDB and Kuzu were not factorized enough and further factorizations produce gains.

Based on early data we're seeing factorized joins DO help with explosion in materialized intermediate tables during deep joins. We're incorporating some of his team's work in stages. The initial support for PACKED logical extend just landed and will be released in a couple of week's time.

## Conclusion

We want to thank The Consensus folks for taking the time to benchmark LadybugDB and write kind words about some of our visualization work (LadybugDB explorer and now Bugscope). In the long run, we believe materializing CSR index on disk, doubling down on novel join algorithms and catching up with DuckDB and Apache Datafusion on traditional join optimizations is the path forward.



