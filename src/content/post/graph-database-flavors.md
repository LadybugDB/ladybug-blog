---
title: "Graph Databases: Why Are There So Many of Them?"
description: "Cut through the confusion around graph databases with a simple mental model of architectures and goals — and see where LadybugDB fits in."
pubDate: "Nov 11 2026"
categories: ["general"]
heroImage: "/img/default.png"
authors: ["team"]
tags: ["graph database", "architecture", "agent context", "ladybugdb"]
---

Graph Databases: why are there so many of them? Exploring this question can be overwhelming. The differences aren't always clear. Let's cut through that confusion using a simple mental model of architectures and goals and then explain where LadybugDB fits in.

## Architecture

- Ease of use (networkx, leveldb, redis)
- Semantically rich (ACID, Best in class storage/query engine)
- Performant (not written in a compiled systems programming language)
- Human centric (as opposed to vibe coded with < 10 authors three of which are AI agents)
- Committee and process (PEP, Architecture Decision Records)

## Objectives

* Be the poster child for a particular language ecosystem (TigerBeetleDB and Zig, CockroachDB and Go)
* Use a two pronged strategy to defend a market (Oracle + MySQL or Neo4j + RelatedDB)
* Research prototype to demonstrate a concept (Umbra)
* Build a business on top of an open source substrate (DuckDB)

## Database for Agent Context

This is a heavily contested recent use case. It's very clear that a server side database that's hard to setup/use is going to be a tough sell in this market. The product needs to be as approachable as networkx in ease of setup.

A number of agent memory companies pursue a dual product strategy. They build an open source version that gets many github stars, launch #1 on product hunt and flood the social media with tech influencer buzz.

They then offer a second, cloud-based "enterprise version" backed by a server-side commercial database with the more desirable architecture. The risk with this setup is that OSS version is a bait and the company's business interest is in keeping its feature set limited so the enterprise version sells.

## LadybugDB Market Position

Some of the larger players such as Neo4j dabbling with agent context is a validation of the market demand. Here's why you should consider Ladybug instead:

* Ease of use: closer to networkx vs hiring a DBA to setup Oracle. Achieved via embedded architecture.
* Semantically rich: Well understood MVCC based Transaction. Hope to run a more rigorous test such as Jepsen. For now, we pass the MVCC bank test (one of the Jepsen components)
* Written in C++. New code written in Rust. Waiting to see where the long term software development goes on a 3-5 year horizon. Betting on a gradual lowering of abstraction.
* Definitely Human centric. Build a bazaar style open for contribution project like Linux. Use AI to stay competitive. Human review of every merged PR. Do not believe in AI consciousness. AI use should be limited to cognition (~1B params). Deterministic Knowledge in an external database with probabilistic indexes.
* Do not believe in process slowing down progress. Recent events around using bureaucracy to shut down projects that have significant user demand is a good case study.
* Objective: build a business on top of an open source substrate. More of an integration, control plane story. Keep the data plane open source.
