---
title: "Ladybug: Spreading its Wings"
description: "Kuzu was archived in October 2025. Explore the major new features in LadybugDB 0.15 and find out why you should migrate today."
pubDate: "Mar 1 2026"
categories: ["release", "migration"]
authors: ["team"]
tags: ["release", "kuzu", "migration", "features", "updates"]
heroImage: "/img/ladybug-so-far/hero.png"
---

If you are a Kuzu user, it is high time you migrate to its vibrant successor, LadybugDB. 

Since Kuzu was officially archived in October 2025, our team forked the repository to ensure the graph database ecosystem continues to thrive under **LadybugDB** with fixing security vulnerabilities and integrating with other data storage systems as a major focus. Over the last few months, we have introduced a massive array of features, performance improvements, and ecosystem expansions that make the [v0.15.0](https://github.com/LadybugDB/ladybug/releases/tag/v0.15.0) release our biggest milestone yet

So what's new?

## Massively Expanded Ecosystem

We've invested heavily in making LadybugDB accessible from anywhere. We now offer first-class bindings for the following languages:

- Python (support for Python 3.14!)
- Node.js
- Rust
- WebAssembly (Wasm)
- Java
- Go
- Swift
- V-lang
- Julia
- Ruby (community)
- Nim (community)

For a deep dive into using these APIs, check out our [Client APIs Documentation](https://docs.ladybugdb.com/client-apis/).

![Ladybug Subgraphs and External Data Architecture](/img/ladybug-so-far/architecture.png)

## Seamless External Database Support

Full support for seamlessly integrating **Arrow**, **DuckDB**, and **Parquet** datasets is here. Rather than migrating or converting your external data into native Ladybug tables, you can simply attach these databases and query them directly! 

Here is how you can use an Arrow source transparently as a node table:

```cypher
CREATE NODE TABLE Employee (id INT64, name STRING, PRIMARY KEY (id))
WITH (storage='arrow://my_arrow_table_id');
```

For parquet, take a look at the *huggingface* [dataset](https://huggingface.co/datasets/ladybugdb/small-kgs)

## Subgraph Support

Creating and querying multiple graph namespaces just got a whole lot easier with true subgraph support. You can now define isolated graphs and switch contexts natively:

```cypher
CREATE GRAPH my_graph;
USE my_graph;
```

You can also CREATE and MERGE nodes without creating a node table first

```cypher
CREATE GRAPH my_graph ANY;
USE my_graph;
CREATE (u:User {name: 'Alice'});
```
Read more about subgraphs in our [doc](https://docs.ladybugdb.com/cypher/data-definition/create-graph/#create-a-subgraph).

## A Better Way to Visualize: Bugscope

Monitoring your data should be pleasant. We've introduced **[Bugscope](https://github.com/LadybugDB/bugscope)**, a react-force-graph-2d based visualizer for LadybugDB.

## Looking Ahead: Future Improvements

We aren't stopping at v0.15. Here's a quick peek at what's coming next:

- **Native SQLite Support**: Seamless querying across SQLite natively in LadybugDB
- **Advanced Arrow & Parquet Optimizations**: Execution layer enhancements to handle even larger, more complex queries over columnar formats efficiently
- **Improved Compatibility with other Graph Databases**: Multi-label support for ANY type graphs without compromising the strict schema and typing

We are incredibly excited about the state of LadybugDB and the incredible community rallying around the project. If you haven't switched yet, there's never been a better time to make the jump!
