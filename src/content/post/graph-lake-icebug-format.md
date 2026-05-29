---
title: "LadybugDB v0.17.0 Release"
description: "LadybugDB v0.17.0 brings graph analytics closer to the lakehouse with Icebug Format v1, better data source integration, leaner indexing, and richer visualization."
pubDate: "May 28 2026"
categories: ["release"]
heroImage: "/img/2026-05-29-graph-lake/ladybug-graph-lake.png"
authors: ["team"]
tags: ["release", "lakehouse", "icebug"]
---

Graph data rarely lives in isolation. It sits alongside Parquet files, relational databases, and object stores — embedded in the same analytical environments where the rest of your data already lives. Yet most graph systems demand you extract it, transform it, load it somewhere else, and manage the copies. That friction is the problem LadybugDB v0.17.0 is built to solve.

This release introduces [Icebug Format v1](https://github.com/Ladybug-Memory/icebug-format), deeper integration across the Arrow ecosystem, smarter SQL pushdown, and a set of targeted improvements that make graph analytics faster, leaner, and more at home inside modern data infrastructure.

> **[Try it in the shell →](https://ladybugdb.github.io/wasm-shell/)** · **[Explore the notebooks →](https://github.com/LadybugDB/ladybug-icebug-notebooks/blob/main/index.ipynb)**

LadybugDB is open source under the MIT license. Thank you to everyone who contributed code, testing, feedback, documentation, examples, and issue reports for this release.

---

## Icebug Format v1

The centerpiece of this release is [Icebug Format v1](https://github.com/Ladybug-Memory/icebug-format) — Ladybug's preferred scalable graph storage format for analytical workloads. We continue to support the more convenient single local file database as our native format.

Icebug is columnar, durable, and designed from the ground up around the shape of graph data. Rather than forcing nodes and relationships through storage paths built for tables or documents, Icebug treats graph structure as a first-class concern. That distinction matters: graph-shaped storage means graph-shaped access patterns, without translation overhead at every layer.

Storage upgrades are now handled **in place**. You no longer need to export a database, import it elsewhere, and reconcile the difference just to move forward to a new format version. Upgrade in place, and keep working. Note that once a database is upgraded to a new version (41 for this release), it cannot be opened by older Ladybug binaries — treat it with the same care you'd apply to any durable storage migration.

---

## Smarter SQL Pushdown

Ladybug is getting better at letting other systems do what they're already good at.

This release improves SQL pushdown into **DuckDB, SQLite, and Postgres**. Instead of pulling data across the boundary and filtering it on Ladybug's side, Ladybug now pushes more of the filtering and selection work down into the source system — where the data lives, where the indexes are, and where that work is cheapest.

The practical result: a cleaner path from existing relational data to graph analytics, with less unnecessary movement in between.

---

## Broader Connectivity via ADBC

LadybugDB v0.17.0 adds **beta support for ADBC** (Arrow Database Connectivity), the emerging standard for connecting analytical systems through the Arrow ecosystem.

ADBC replaces a growing list of one-off integrations with a single, standard interface. For Ladybug, that means more data sources become reachable without bespoke connectors — and it means applications that already produce Arrow tables can use Ladybug as a Cypher-based graph engine over that memory directly. No separate graph database format required; Arrow tables become graph data, and Cypher runs over them immediately.

This is still early, but the direction is clear: fewer integration seams, more reachable data.

---

## Leaner Indexing

Indexes received a lot of practical attention in this release, with a focus on giving users real control over the tradeoff between space and lookup speed.

- **Disabling the default hash index** is now supported for workloads where space efficiency matters more than fast primary-key lookup — a meaningful option for large analytical graphs where index storage is a real cost.
- **Optional ART indexing** (borrowed from DuckDB's approach) enables primary-key range scans in Ladybug for the first time. When fast, precise lookups are the priority, opt in. When they're not, opt out and keep the footprint lean.

---

## Deeper Icebug Integration

The integration between Ladybug and Icebug is substantially deeper in v0.17.0.

Ladybug can now read relationship tables directly into Icebug memory layouts and run analytics with zero-copy-style paths wherever possible. This eliminates a significant source of overhead: data no longer needs to be moved and re-shaped before analysis can begin.

Work on Arrow relationship tables, CSR layouts, relationship scans, and row IDs improved the efficiency of moving relationship data through Ladybug, Icebug, and the broader Arrow ecosystem. The Python layer now exposes these capabilities more directly, making the full path accessible from notebooks and scripts without low-level plumbing.

---

## Bugscope Visualization

[Bugscope](https://github.com/LadybugDB/bugscope) now integrates directly into the Ladybug and Icebug stack.

This release adds support for **in-process analysis** alongside **hierarchical clustering visualization**, making it practical to inspect, compare, and understand graph structure visually as part of the same analytical workflow — not as an afterthought that requires exporting to a separate tool.

Fast analytics tell you what the answer is. Seeing the shape of the graph helps you understand *why*.

---

## Python Improvements

Two meaningful improvements landed for Python users in this release.

**Memory release on result close.** Query results now release their memory when the result object is closed, rather than waiting for the connection or database to be garbage collected. For long-running processes and larger analytical workloads, this makes memory behavior predictable and controllable.

**C API-backed Python bindings.** This release adds Python bindings built on Ladybug's C API — the same foundation used by the other language backends. The existing pybind-based package remains the default, but the C API path establishes a more consistent cross-language base that may become the default in a future release.

Higher-level language integrations can now use **strongly typed schema declarations** that translate declaratively to Ladybug Cypher. Application code can keep type-safe schema definitions in the host language while targeting Ladybug's graph model underneath. See this [Python example](https://github.com/adsharma/fquery/blob/main/examples/ladybug_demo.py) for a concrete walkthrough.

---

## Remote Storage and Hugging Face Datasets

Real lakehouse data lives outside the local filesystem — in object storage, behind HTTP endpoints, and increasingly on Hugging Face. This release strengthens that story on several fronts.

Remote and object storage support improved with virtual file system work, HTTP-style access paths, and Xet-backed storage. Ladybug now supports **accessing Hugging Face datasets directly through the Xet protocol**, making it possible to work with published graph datasets from remote storage without a local download step:

```
lbug -i xet://datasets/ladybugdb/small-kgs/main/kg_history/icebug-disk/schema.cypher
```

Published datasets are available at [huggingface.co/ladybugdb/datasets](https://huggingface.co/ladybugdb/datasets).

Parquet handling also improved, with better relationship table scans and prefetching — making larger analytical datasets faster and more practical to work with at scale.

---

## Other Notable Work

- **C API and Arrow improvements**: Arrow table registration, better relationship-table support, and broader embedding capabilities for tools and language bindings built on Ladybug.
- **Correctness and reliability**: Significant work on joins, MERGE behavior, recursive scans, visibility checks, filters, relationship scans, Windows builds, packaging, and release automation. Less visible than new features, but the foundation everything else rests on.
- **Notebooks**: The new [Ladybug Icebug notebooks](https://github.com/LadybugDB/ladybug-icebug-notebooks/blob/main/index.ipynb) repository provides hands-on Jupyter walkthroughs for learning the Icebug workflow end to end.
- **Benchmarks**: We've seen noticable improvements in memory usage and runtime for python users due to improved memory management. Many types of count queries are now served from node group metadata instead of scanning rows.

---

## Summary

LadybugDB v0.17.0 is a release about **less copying, better integration, and making graph analytics feel at home inside modern data systems**.

The key additions:

- [Icebug Format v1](https://github.com/Ladybug-Memory/icebug-format) with in-place storage upgrades
- Improved SQL pushdown into DuckDB, SQLite, and Postgres
- Beta ADBC support for broader, standard data connectivity
- Cypher over Arrow memory — no separate graph format required
- Flexible indexing: disable the hash index to save space, or opt into ART-based range scans
- Zero-copy-style relationship analytics via deeper Icebug integration
- Predictable Python memory release; C API-backed bindings
- Typed-schema-to-Cypher workflows for application code
- Hugging Face dataset access via Xet
- [Bugscope](https://github.com/LadybugDB/bugscope) visualization integrated into the analysis stack
- [Notebook-based Icebug tutorials](https://github.com/LadybugDB/ladybug-icebug-notebooks/blob/main/index.ipynb)