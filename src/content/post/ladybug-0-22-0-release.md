---
title: "LadybugDB v0.22.0 Release - Halloween Edition"
description: "LadybugDB v0.22.0 brings CHECKPOINT hardening, lookup joins, lower query latency, Snowflake and Databricks lakehouse support, DuckDB 2.0.0, SQLite replication, and Bugscope updates."
pubDate: "Oct 26 2026"
categories: ["release"]
heroImage: "/img/2026-10-26-halloween/ladybug-halloween.png"
authors: ["team"]
tags: ["release", "checkpoint", "query optimizer", "lakehouse", "snowflake", "databricks", "duckdb", "sqlite", "bugscope"]
---

The 0.22.0 release is arriving a few days early. It's one of the biggest monthly releases of 2026 in terms of sheer volume of commits. Many of them are much needed bug fixes and perf enhancements. As you can see @zahariash and Claude Opus 5.5 are top committers this time around. We also have 6 other first time committers. Thank you all for making ladybug much better than it was a month ago.

![Commits over time](/img/2026-10-26-halloween/commits-over-time.png)

![LadybugDB v0.22.0 Halloween Release](/img/2026-10-26-halloween/ladybug-halloween.png)

## Data Integrity

The largest area that received attention was around CHECKPOINT operations. One of the most basic things a database does is to not lose data. It makes a promise that the data was received at COMMIT. But only the WAL record is written to disk. Data is still buffered in RAM until we do a CHECKPOINT.

A large number of bugs were fixed in this general area, so a failed CHECKPOINT doesn't corrupt the data and the database continues to operate smoothly.

We also hope to turn on @aikins01's previous work on concurrent writes in the next release by default. You can flip a flag and test concurrent writes in this release as well, but the feature is disabled by default.

## Better Plans

If you read the Kuzu VLDB 2025 paper, a few query plans around Hash Joins, Factorized Joins and ASP-Joins that were discussed. These were all novel algorithms that improved the state of the art. But applying those plans to simple queries such as the ones in Prashanth Rao's [benchmark](https://github.com/hyperdimensionalcomputingai/graph-benchmark-ldbc) was a mistake.

We noticed that some of the competing databases were doing INLJ (index nested loop joins), which is essentially two for loops to perform a join. So we added lookup-joins (a variant of INLJ) to our repertoire.

Also, even simple queries like `RETURN 1` had a constant overhead that woke up all the threads on a multi-core machine. We optimized many of them away, so the minimum query latency is much lower than previous releases.

While there are opportunities to further reduce this, our goal is not to compete with memcache/KV like systems on latency. Goal is to have < 3ms latency for easy queries and really focus on hard queries where people choose us.

## Snowflake and Databricks

Previously we would talk to these catalogs through duckdb's iceberg/unity catalog support. But we found that it was easier to use ADBC drivers using columnar.tech's dbc driver manager.

Both of these lake houses will be supported in this release and available on the respective marketplaces for a supported version.

## DuckDB 2.0.0

DuckDB made a major release just 5 days ago with improve asyncio, variant support and many query engine optimizations.

If you're a duckdb user and want to query the tables via ladybug cypher, it should work as well as any other graph query to SQL translator.

## SQLite Replication

We now link a copy of sqlite directly into the extension instead of reusing duckdb's sqlite extension. As a second feature, it's possible to setup replication from sqlite row oriented tables to ladybug columnar tables.

We envision coding agents that use sqlite such as pidotdev to use ladybug's columnar storage for historical storage due to efficiency and context/knowledge graph support.

## Bugscope 0.22.0

Bugscope has been updated as well. You can now use a combination of CLI and GUI to query graphs as large as 90 million nodes, 750 million edges.

You can also query remote ladybug instances over ADBC.

Once again, thank you for being a part of the ladybug community and contributing to its growth. Happy Halloween!
