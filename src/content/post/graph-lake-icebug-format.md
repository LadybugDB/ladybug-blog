---
title: "LadybugDB v0.17.0 Release"
description: "LadybugDB v0.17.0 brings graph analytics closer to the lakehouse with Icebug Format v1, better data source integration, leaner indexing, and richer visualization."
pubDate: "May 28 2026"
categories: ["release"]
authors: ["team"]
tags: ["release", "lakehouse", "icebug"]
---

LadybugDB v0.17.0 is a big step toward a graph-native lakehouse.

The theme of this release is simple: keep graph-shaped data close to where it already lives, avoid unnecessary copying, and make Ladybug and Icebug faster and easier to use across real-world storage systems. Graph analytics should not require a maze of exports, imports, and temporary copies before the useful work can begin.

## Graphs Meet The Lakehouse

This release lays the foundation for storing and analyzing graph data in a lakehouse-friendly way.

The centerpiece is Icebug Format v1. Icebug is Ladybug's graph storage format for analytical workloads: durable, columnar, and designed around the shape of graph data. Instead of squeezing relationships and nodes through storage paths built for other kinds of data, Ladybug can now work with a format that treats graph structure as a first-class concern.

That matters because graph data is often already part of a larger analytical environment. It may live beside Parquet files, relational databases, object storage, or other data systems. Ladybug v0.17.0 moves us closer to running graph analytics where the data already is.

## Better Pushdown Into SQL Engines

Ladybug is also getting better at letting other systems do the work they are already good at.

This release improves SQL pushdown into DuckDB, SQLite, and Postgres. In practice, that means Ladybug can push more filtering and query work down into the source system instead of pulling too much data across the boundary and sorting it out afterward.

For users, the result is a cleaner path from existing data to graph analytics. You can keep using the systems that already hold your data, while Ladybug focuses on the graph-shaped parts of the problem.

## More Data Sources Through ADBC

Ladybug v0.17.0 adds beta support for ADBC, the Arrow Database Connectivity standard.

ADBC gives Ladybug a broader, more standard way to connect to analytical data sources through the Arrow ecosystem. This is still early, but it is an important direction: more data sources, fewer one-off integrations, and a cleaner foundation for future connectors.

## Leaner Indexing

Indexing received a lot of practical attention in this release.

Ladybug can now disable the default hash index in cases where saving space matters more than fast primary-key lookup. That is especially useful for large analytical graphs, where indexes can take meaningful space and are not always needed for the workload.

There is also optional support for using DuckDB's ART indexing approach for primary-key lookups and range scans. The goal is flexibility: use less space when indexes are not needed, or opt into stronger lookup behavior when they are.

## Deeper Icebug Integration

Icebug integration is much deeper in v0.17.0.

Ladybug can now read relationship tables into Icebug memory layouts and run analytics with zero-copy-style paths where possible. This reduces the overhead of moving data around before analysis can start.

There was also important work around Arrow relationship tables, CSR layouts, relationship scans, row IDs, and Python exposure. The plain-English version: relationship data is becoming much more efficient to move through Ladybug, Icebug, and the broader Arrow ecosystem.

## Bugscope Visualization

Bugscope now fits more naturally into the Ladybug and Icebug stack.

This release supports in-process analysis with Ladybug and Icebug, along with hierarchical clustering visualization. That makes graph structure easier to inspect, compare, and understand visually. Faster analytics are useful; being able to see the shape of the result makes them more useful.

## Python Improvements

Python also moved forward in this release.

Query results now release memory when the result is closed, instead of waiting until the connection or database object is garbage collected. For long-running Python processes and larger analytical workflows, that makes memory behavior more predictable.

This release also adds support for Python bindings built on top of Ladybug's C API. The existing pybind-based Python package remains the default today, but the C API path is important because it is the same foundation used by the other language backends. Over time, that gives Ladybug a more consistent cross-language base, and it may become the default Python path in the future.

## Other Notable Work

Remote and object storage support improved, including virtual file system work, HTTP-style access paths, and Xet-backed storage. This strengthens the lakehouse story because real lakehouse data often lives outside the local filesystem.

Parquet handling also improved, including relationship table scans and prefetching. These changes make larger analytical datasets faster and more practical to work with.

The C API and Arrow paths gained new capabilities, including Arrow table registration and better relationship-table support. These are the kinds of lower-level improvements that make Ladybug easier to embed in other tools and language bindings.

There was also a lot of correctness and reliability work around joins, MERGE behavior, recursive scans, visibility checks, filters, relationship scans, Windows builds, packaging, and release automation. These are less flashy than new features, but they are what make the bigger architectural work usable.

## The Short Version

LadybugDB v0.17.0 moves graph analytics closer to the lakehouse. It introduces Icebug Format v1, improves pushdown into DuckDB, SQLite, and Postgres, adds beta ADBC support for broader data access, gives users more control over indexing and storage size, deepens the Icebug integration path, improves Python memory behavior, and makes graph structure easier to explore through Bugscope visualization.

It is a release about less copying, better integration, and making graph analytics feel at home inside modern data systems.
