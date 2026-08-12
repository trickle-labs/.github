# Trickle Labs

**PostgreSQL as the data platform.**

## What we believe

Most organisations today run half a dozen specialised databases alongside the one they started with. A graph database for relationships. A streaming engine for real-time analytics. A message broker for events. A search engine for free text. A vector store for AI. Each system comes with its own deployment model, its own backup strategy, its own failure modes, and its own team that knows how to operate it. The data that matters most — the data that drives decisions — is scattered across all of them, and keeping it consistent is a full-time job that never quite gets done.

We think there is a better path. PostgreSQL is already the world's most advanced open-source relational database. It has been built, tested, and trusted by thousands of contributors over more than three decades. It handles transactions, replication, security, backups, and monitoring with battle-tested maturity that no young project can match. What it has lacked, until now, is the ability to do all the things those specialised systems do — without leaving the database.

Trickle Labs exists to close that gap. We build PostgreSQL extensions and companion tools that bring streaming, knowledge graphs, property graphs, messaging, and AI-native retrieval directly into the database you already run. No pipelines to operate. No data to duplicate. No new infrastructure to learn. One database, one backup, one operational model — and a dramatically wider set of things you can do with it.

## What we build

Everything we release is open source under the Apache 2.0 license. Our projects compose together but each one stands on its own.

### Streaming tables that stay fresh automatically

[**pg-trickle**](https://github.com/trickle-labs/pg-trickle) is the foundation of much of what we do. It adds *stream tables* to PostgreSQL — tables defined by a SQL query that keep themselves up to date as the underlying data changes. If you have ever wished a materialized view would just refresh itself, instantly and efficiently, this is that. When one row changes in a million-row source table, pg-trickle processes one row's worth of computation. Not a million. It uses differential dataflow — implemented as a native Rust extension that lives inside PostgreSQL and uses nothing outside it.

Stream tables can depend on other stream tables. A single write to a base table can ripple through a graph of derived views, each updated in the correct order, each doing only the work proportional to what actually changed. The system figures out scheduling automatically: you declare how fresh you need your consumer-facing tables to be, and the upstream plumbing inherits that cadence without any manual configuration.

### Rules that turn changing data into durable work

[**pg-react**](https://github.com/trickle-labs/pg-react) is a PostgreSQL-native rule engine for turning changing data into durable decisions and work. You define conditions as ordinary SQL views, and pg-react remembers when a matching row appears, changes, disappears, and later reappears. Built on pg-trickle's incremental view maintenance, it gives each match a stable identity and records its lifecycle as durable, queryable state — so a risk threshold crossing, overdue invoice, inventory shortage, or policy violation becomes an event that can be acted on exactly when it matters.

Consequences can run as typed PostgreSQL functions or be handed to external systems through a transactional outbox. Activations, retries, leases, execution history, and rule versions all live in PostgreSQL, making the entire path from source data to completed action inspectable with SQL. pg-react also supports maintained derived facts and recursive reasoning, extending the same model from reacting to changes to continuously deriving new knowledge from the data already in the database.

### Messaging without a message broker

[**pg-tide**](https://github.com/trickle-labs/pg-tide) gives PostgreSQL a built-in messaging backbone. It implements the transactional outbox pattern: you publish events inside the same database transaction as your business logic — no dual-writes, no distributed transactions, no chance of messages getting lost or duplicated. When you are ready to fan out to Kafka, NATS, Redis Streams, or any of fifteen other platforms, a lightweight relay binary bridges the gap with exactly-once delivery semantics. Pipeline configuration lives in PostgreSQL itself and hot-reloads without restarting.

### A knowledge graph engine inside PostgreSQL

[**pg-ripple**](https://github.com/trickle-labs/pg-ripple) turns PostgreSQL into a standards-compliant knowledge graph store. You can model data as a web of connected facts — entities, relationships, and properties — and query it with SPARQL, validate it with SHACL quality rules, and reason over it with Datalog and OWL. It passes 100% of the W3C conformance suites for SPARQL 1.1, SHACL Core, and OWL 2 RL — the industry benchmarks for correctness.

Knowledge graphs are particularly powerful for AI applications. pg-ripple combines graph traversal with vector similarity search in a single query, so you can ask "find papers similar to this topic, authored by people in Alice's network" without stitching together three different systems. It generates structured, token-efficient prompts for language models directly from graph queries, and its entity resolution pipeline deduplicates records across sources without exposing raw personal data.

### Property graphs with native performance

[**pg-eddy**](https://github.com/trickle-labs/pg-eddy) brings labelled property graph storage into PostgreSQL via a custom storage engine. Instead of simulating graph traversal with index lookups on regular tables, pg-eddy stores adjacency information directly alongside node data on disk. The result is O(degree) per-hop traversal — meaning that following a relationship from one entity to the next takes time proportional only to how many relationships that entity has, not to the size of the entire graph. It supports openCypher, the most widely-used property graph query language, with 100% compliance against the official Technology Compatibility Kit.

### Safe schema evolution for streaming pipelines

[**pg-aqueduct**](https://github.com/trickle-labs/pg-aqueduct) is the migration tool for teams that operate pg-trickle in production. Adding a column, changing an aggregation, or restructuring a pipeline of stream tables today requires careful manual work to avoid downtime and data loss. pg-aqueduct computes a safe execution plan automatically — preserving accumulated state wherever the mathematics allow, enforcing topological order across dependencies, and offering rollback when things go wrong. It occupies the same role for streaming pipelines that schema migration tools occupy for relational schemas and infrastructure.

### A knowledge compiler for documents

[**riverbank**](https://github.com/trickle-labs/riverbank) is a Python application that turns raw documents — Markdown, PDF, HTML, DOCX — into a governed, queryable knowledge graph. It uses language models to extract structured facts from unstructured text, attaches confidence scores and source citations to every claim, validates the output against quality contracts, and writes the result into pg-ripple. When a source document changes, only the knowledge derived from that document needs to be recompiled — not the whole corpus. The result is a living knowledge base that stays current as the world changes, with full provenance for every fact it contains.

### Visual exploration without query languages

[**Moire**](https://github.com/trickle-labs/moire) is a web application that makes knowledge graphs navigable for people who are not database engineers. Point it at any SPARQL endpoint and it introspects the data automatically — discovering what types of entities exist, which relationships are available, and how to label things meaningfully. You explore by clicking, filtering, and following connections. The interface builds itself from whatever data it finds, with no configuration required. Think of it as a newspaper for structured data: you scan headlines, pick a section, read a story, follow a link to something related, and backtrack whenever you want.

### Query language interoperability

[**rs-polygraph**](https://github.com/trickle-labs/rs-polygraph) is a Rust library that transpiles openCypher and ISO GQL queries into SPARQL algebra. It enables any SPARQL-compliant engine to accept property graph queries without reimplementing execution logic — bridging the two major families of graph query languages with a single translation layer.

## Why PostgreSQL

We chose to build on PostgreSQL for reasons that go beyond technical preference. PostgreSQL is genuinely open — not open-core, not source-available with restrictions, but developed by a global community with no single corporate owner. It has an extension architecture that is unmatched in the database world: you can add entirely new storage engines, query languages, data types, and index structures without forking the core. Its operational toolbox — WAL replication, point-in-time recovery, connection pooling, role-based security, monitoring hooks — is the product of three decades of production use. And it runs everywhere: on your laptop, in a container, on managed services from every major cloud provider, on Kubernetes via CloudNativePG, and on bare metal in your own data centre.

When you build on PostgreSQL, every operational practice your team already knows continues to work. The same `pg_dump` that backs up your application tables backs up your knowledge graph. The same monitoring that watches your transaction rate watches your stream tables. The same security policies that protect your customer data protect your graph queries. There is no second system to learn.

## Who we are

Trickle Labs is a small team building open-source data infrastructure. We believe that the complexity of the modern data stack is not an inevitable consequence of ambitious requirements — it is an accident of history, and PostgreSQL is the platform where that history can be rewritten. Every project we ship is Apache 2.0 licensed. We welcome contributions, feedback, and design discussions on any of our repositories.

## Get involved

- Browse our [repositories](https://github.com/orgs/trickle-labs/repositories)
- Open an issue on any project to ask a question or suggest an improvement
- Read the documentation — every repository includes a detailed README, architecture docs, and a roadmap
- Try `pg-trickle` or `pg-ripple` on your existing PostgreSQL 18 installation — they work alongside your current data without requiring any migration
