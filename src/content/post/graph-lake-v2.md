
---
title: "LadybugDB v0.21.0 Release"
description: "LadybugDB v0.21.0 brings correctness, stability and performance improvements with GraphLake, SQL pushdown, and lakehouse integrations."
pubDate: "Sep 30 2026"
categories: ["release"]
heroImage: "/img/2026-09-30/graph-lake-v2.png"
authors: ["team"]
tags: ["release", "lakehouse", "graphlake"]
---

This is more of a correctness, stability and performance improvement release. New features: Graph Lake and Improved support for SQL pushdown and external catalogs and Lakehouse table formats.

### Storage, WAL, checkpoints, indexes

A failed/partial checkpoint no longer leaves the disk in a corrupted state where it can't be recovered. Hash PK corruption is handled via runtime exception rather compile time `DASSERT` which is compiled out on release builds.

### Correctness

We now pass `TSAN` and `UBSAN`. While this doesn't prove that we don't have concurrency bugs, it improves the confidence in the quality of the code.

Many improvements that fix problems where previous releases silently returned wrong results

### Planner and optimizer

Support for more query patterns that are now handled with optimized physical operators instead of materializing millions of rows. Net result: leadership position across LDBC SNB unofficial benchmarks: the 14 complex queries and 30 Prashanth Rao queries. Much improved results on LSQB as well.

### External Catalogs and SQL Pushdown

We now support scanning a SQL catalog by convention. As long as the tables and columns are named a certain way, you can attach and start querying tables by simply attaching and using cypher. The queries are auto translated to SQL and executed efficiently by the underlying planner. More [query patterns](https://github.com/LadybugDB/extensions/blob/main/duckdb/test/test_files/sql_pushdown.test) including recursive CTEs are supported.

When you do `export database '/path/to/backup`, we now use the icedisk format for export. This format can be queried via a [duckdb extension](https://github.com/Ladybug-Memory/duckdb-icedisk-extension) or the exported data can be moved to object storage and queried remotely using `hf://..` or `xet://...` URLs.

We are inspired by the idea of using the catalog of a SQL or Cypher database instead of JSON files on object storage. This is the basis of GraphLake (Iceberg tables + Icebug-Disk). It's only fitting to add support for DuckLake via a new [ducklake extension module](https://github.com/LadybugDB/extensions/blob/main/ducklake/test/test_files/ducklake.test).

```
ATTACH 'users.ducklake' as dl (dbtype ducklake);
MATCH (a:dl.person)-[b:knows_rel]->(c:dl.person) RETURN count(*);
```

Improved documentation on how to use Ladybug with Iceberg tables and on vendor clouds such as [Snowflake](https://docs.ladybugdb.com/integrations/snowflake/) and Databricks (coming soon!).

### Postgres Integration

Postgres is everyone's favorite OLTP database. The Postgres core team has [chosen](https://gdb-engines.com/blog/graph-technology-august-round-up/) to defer the implementation of PGQ (a graph query language) to the next release, which is likely a year away. In the meanwhile, you can continue to use the `pg_ladybug` [extension](https://github.com/LadybugDB/pg_ladybug).

The pg_ladybug approach is similar to `pg_duckdb` in that it brings in columnar storage which is much faster than running PGQ over heap storage. Unlike `pg_duckdb`, `pg_ladybug` also supports local NVMe storage in addition to object storage.

### GraphLake Vision

To understand how all these pieces fit together, we put together this explainer.

1. How Iceberg works today
2. DuckLake design (supported in 0.21.x)
3. Icebug Disk design (very similar to DuckLake. Replace SQL with Cypher)
4. Where we are today
5. Where we want to go

![GraphLake Vision](/img/2026-09-30/graph-lake-v2.png)
