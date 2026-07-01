
---
title: "Ladybug Flying Solo"
description: "LadybugDB with its innovations is more than a kuzu fork"
pubDate: "July 1 2026"
categories: ["benchmarks", "kuzu"]
heroImage: "/img/2026-07-01-solo/ladybug-solo.png"
authors: ["team"]
tags: ["benchmarks", "kuzu", "legcacy", "successor", "security", "graph database"]
---
With the release of 0.18.0, LadybugDB is making a mark in the world of embedded graph databases. While it's built on top of an excellent technology base provided by the KuzuDB project, we argue here that it's more than a Kuzu fork with innovations that should be evaluated on their own merit.

Our objective is to convince you that you shouldn't really be using the archived Kuzu project or its lightly modified forks for security reasons. When it comes to competing graph databases, make your choice based on merits and not aggressive marketing tactics, SEO or sales pitches.

Kuzu was first released in 2022 and had not one, but two VLDB papers. It took the database community by storm by innovating in a number of dimensions including a practical, strongly typed version of Cypher, factorized join algorithms and excellent performance.

When Apple acquired the key people on the project, there was a flurry of activity to claim the excellent reputation built up by the project and its leaders. There were forks ranging from one line README update to a few bug fixes and a single feature fork. We are intentionally ignoring any private/closed-source forks which may or may not exist.

A number of competing databases have responded to the news by publishing migration guides and in some cases used inbound marketing techniques also known as SEO.

## Why you shouldn't use Kuzu

We're big fans of the erstwhile Kuzu team here and very thankful for their contributions. What is being discussed here is the simple reality of living in the age of Mythos. Kuzu team made many good technical calls/judgements. But..

Any large code base has bugs. Kuzu had about 100k lines of code and 3x as many lines of third party code. This is a giant risk for a C++ project.

This risk exists not just for Kuzu, but all the lightly modified forks.

## Why should you use Ladybug

We've fixed most known/reported bugs. That's not to say that new bugs didn't come in. The risk is always there. But we're here fixing them as they get reported, while Kuzu stays archived.

The rest of the blog post is about recent innovations.

## Billion Scale Graphs

With Kuzu and its design objective of having the fastest join algorithm for deep graph traversals, you simply couldn't import a billion scale graph due to space amplification. Import a 1GB parquet into Kuzu and it'd consume between 10-20GB depending on the distribution of keys and configuration.

With this release, we can demo a [10GB zstd compressed database](https://huggingface.co/datasets/ladybugdb/wikidata-20260401/tree/main) that you can query with less than 8GB of RAM to get any triples you want. After decompressing it consumes 20GB on disk. This demo was simply not practical with kuzu due to design choices.
```
lbug> call disk_size_info() return *;
┌────────────┬───────────────────────────────────────────┬───────────┬─────────────┐
│ category   │ name                                      │ num_pages │ size_bytes  │
│ STRING     │ STRING                                    │ UINT64    │ UINT64      │
├────────────┼───────────────────────────────────────────┼───────────┼─────────────┤
│ header     │ database_header                           │ 1         │ 4096        │
│ catalog    │ catalog                                   │ 1         │ 4096        │
│ metadata   │ metadata                                  │ 783       │ 3207168     │
│ node_table │ wikidata_node                             │ 1440241   │ 5899227136  │
│ index      │ wikidata_node.name_index:tree             │ 1275258   │ 5223456768  │
│ node_table │ edge_meta                                 │ 90        │ 368640      │
│ index      │ edge_meta._PK:hash_index_headers          │ 2         │ 8192        │
│ index      │ edge_meta._PK:disk_array_headers          │ 3         │ 12288       │
│ index      │ edge_meta._PK:primary_slots               │ 512       │ 2097152     │
│ node_table │ edge_types                                │ 0         │ 0           │
│ rel_table  │ wikidata_rel:wikidata_node->wikidata_node │ 2140331   │ 8766795776  │
│ free_space │ free_pages                                │ 679577    │ 2783547392  │
│ total      │ file_total                                │ 5536799   │ 22678728704 │
└────────────┴───────────────────────────────────────────┴───────────┴─────────────┘
```
## Choice of Indexing

Very few databases, especially columnar databases use hash indexing by default. But Kuzu chose to do so because it was a key part of the Accumulate-Semijoin-Probe (ASP) join. This involved a trade-off: space amplification and range queries. Hash indexes are great for probes, but do not maintain ordering. Also Kuzu didn't implement indexing on columns other than the primary key.

Further the hash indexing was so deeply ingrained in the kuzu code base that it took us two release cycles to make it optional. So here we are!

You can now disable default hash indexing and choose Adaptive Radix Tree (ART) indexing as a second choice. Some queries are faster, some are slower. You choose the type of index based on the workload, like all normal databases do.

We implemented UDFs `disk_size_info()` and `show_indexes()` to display the space consumed by indexes.

One of the reasons why kuzu was outperformed by DuckDB and LanceDB on simpler queries that didn't involve ASP joins was related to this.

## Stable Storage Format

Kuzu's storage format was not stable. Each release you had to export/import databases to be able to use new features. We have since implemented a Typespec -> C++ code generator that makes it work like protobuf, but simpler and more suitable for WAL records.

When you add a field to an existing record (the most common change), we can now upgrade in place and new versions will handle old records just fine because of the persisted record size on disk.

## Platform Coverage

We support many more languages (9 and counting) and platforms (os * cpu * glibc). Had to request a disk size upgrade from pypi.org so we could support so many platforms.

Also curl + bash, github releases, homebrew, debian, rpm, nix and FreeBSD packaging are supported.

## Graph Lake

Generalizing storage beyond the graph native storage (which we highly recommend) is a key area of focus. As a part of this effort, we now support Icebug-Disk and Icebug-Memory (Parquet and Arrow based graph storage) and lake house architecture where your relational data can be queried in place via pushed down SQL.

Any data source with a duckdb extension or columnar's ADBC protocol and the `dbc` tool are supported.

Neo4j virtual graph and puppygraph implement similar functionality, but lack the graph native columnar storage Ladybug has. We also use best practices pioneered by DuckLake - use a database for metadata instead of YAML files.

We have not done extensive benchmarking against competition. But it's so simple to use that you can figure it out with only basic documentation.

## Improved Write Performance

Kuzu did single threaded writes to disk and used 2 fsync() calls per transaction. So the number of small atomic writes you could do was limited by `1/(2 * fsync_latency)`. If your fsync() took 4 ms, you could only do 125 transactions/sec.

We updated the algorithm to do one fsync() per transaction, doubling the throughput. Further, you can now enable multi threaded writes where multiple threads can perform writes, but only one thread can write to the log so we maintain the MVCC guarantees. But that one thread now becomes the bottleneck. So we'd still be doing 250 transactions/sec.

Many databases do group commits. The idea is to group N transactions and block the callers until the group as a whole is written to durable storage that is the write-ahead-log (WAL). After this change, we measured 6k transactions/sec in one configuration.

The part before the group commit was work by Vela Partners that was ported to Ladybug by Logan Powell in the last release.

## NaviX fully implemented

NaviX is a novel adaptive algorithm for graph-vector search. A big chunk of it was already implemented, but there were pieces missing.

We completed the implementation as described in the paper. NaviX is now the default vector search algorithm. We invite you to test/benchmark it against previous versions and against other impementations.

## Subgraphs and Neo4j compatible Open Type Graphs

This is a novelty we shipped in a previous release. A lot of our users come from neo4j and are surprised by how simple queries such as CREATE without a table don't work or they can't attach multiple labels to a node.

Kuzu took an uncompromising strongly typed cypher stance. We think it was a wise move. Our competition is trying to adopt it, but it feels more like an afterthought.

We want to continue the Kuzu tradition by default. But by allowing open type subgraphs, we allow the creation of a second **separate** universe. These two graphs don't mix. So no poorly defined, badly specified cross graph queries exist.

Like other major databases, we now have a JSON type in the core. You don't need an extension.

## Benchmarking

We've seen some benchmarks where Kuzu outperforms Ladybug on some queries. It's consistent and not related to any particular type of query.

One thought was that it was caused by feature bloat, indexing trade-offs or worse clueless developers vibe coding and committing to Ladybug repo.

We've since debugged this issue a bit and attribute it to lack of reproducible builds due to self hosted infra used by Kuzu CI.

We can recover some of the performance by using profile guided optimization (PGO), but decided not to ship it due to the cost of maintenance.

If you're in a performance sensitive environment, we encourage you to do your own PGO based on the workload and if you feel like it build binaries with same RedHat gcc-14.x used by the last Kuzu build before it was archived.

We think it's much better to focus on large query optimization wins vs minor build env differences. Full benchmark data [here](https://gist.github.com/adsharma/b1815293b194533525c53d1dd93413c6).

## Counts and Scans

One feature that's unique to Ladybug and Kuzu is relationship (REL)tables. After all, that's what makes it a graph database. Other larger columnar databases such as DuckDB and ClickHouse can always spend more resources optimizing their native tables (which are simialr to Ladybug Node tables).

Our inclination is to focus on optimizing REL tables. Who knows, one day we may just adopt another market leading open source columnar table format as our preferred node table format.

Among the improvements we did over kuzu on the following queries: 

```
MATCH ()-[r:Owns]->() RETURN COUNT(*);  // 13x faster
MATCH (a:Item)-[r:Owns]->(b:Item) RETURN a.rowid, b.rowid;  // 50-60x faster
```
Since the data is stored in columnar CSR storage and the result is being requested via Arrow memory, why bother with all the intermediate steps? Zero-copy is the way to go!

This matters a lot when running Icebug algorithms for graphs stored in Ladybug. Fine print: requires you to sort the data before hand and store it only in `fwd` direction. This allows us to use multi-core morsel driven parallelism and avoid a merge sort (which forces a copy).
## Summary

There is enough in here to suggest that this is not just a lightly modified fork of Kuzu. We invite you to make your own assessment on these innovations. While we're proud to carry the Kuzu torch forward, we want to be seen as more than just a Kuzu fork.

About Ladybug not showing up on a search for `kuzudb` on major search engines: it's a sad state of affairs. The best technology doesn't always win, but smart people figure out where good work is happening. If you choose to, you can help and vote with a link to ladybugdb.com with keywords like kuzu, ladybug and embedded graph database.



