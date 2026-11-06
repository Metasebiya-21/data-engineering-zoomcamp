# 3. Databases

> This section covers the storage engines that sit under almost every data system: what a database is, the relational model and SQL, keys and schemas, NoSQL families, OLTP, vector databases for RAG, managed databases, the CAP theorem, and short profiles of popular engines (MongoDB, PostgreSQL, Redis, Elasticsearch/OpenSearch, etcd, Cassandra, HBase, CouchDB, Presto). It ends with query optimization and ER diagrams. In interviews for a data-intensive backend or lead data engineering role, this is where the hardest follow-up questions come from: how indexes work, what ACID really guarantees, which isolation level you run, how you would pick between PostgreSQL, MongoDB and Redis for a feature, and what you give up when a system is distributed. Know the trade-offs, not just the definitions.

## Database
*Source: [IBM — What is a Database?](https://www.ibm.com/think/topics/database)*

**In one line:** A database is a managed system (storage plus a DBMS) for storing, organising, securing and querying collections of data, so many users and applications can share one trustworthy copy.

**Key points**
- A database system has two parts: the storage that holds the data and the database management system (DBMS) that people and apps use to define, query, change and secure it. A spreadsheet is a single file, not a database.
- Main families: navigational (hierarchical trees and network/pointer models, now mostly historical), relational (tables queried with SQL), NoSQL (document, key-value, wide-column, graph), object-oriented, vector, and cloud-hosted variants. Multimodel and in-memory databases are further categories.
- Most deployments separate the application tier from the database tier so each can scale and fail independently.
- A data model (conceptual design) leads to a database model and a schema (the concrete rules: tables, types, relationships).
- Databases matter for usability (one curated store), integrity (central rules instead of drifting copies) and security/compliance (role-based access, auditing, GDPR/HIPAA).
- Choosing one depends on data type, purpose, performance needs, price and scalability (vertical: bigger machine; horizontal: more machines).

**Compare / don't confuse with:** Database (fast capture, transactions, point queries) vs. data warehouse (cleaned, integrated data for analytics) vs. data lake (cheap raw storage of any format) vs. lakehouse (lake storage plus warehouse-style query engine and governance).

**Interview angle**
- *Q: How do you choose a database for a new service?* — Start from the access patterns and consistency needs, not the brand: transactional data with relationships and invariants goes to a relational database; flexible, nested, document-shaped data with simple access paths can suit a document store; hot, short-lived data goes to an in-memory store; relationship-heavy traversals suit a graph. Then check scale, operational skills on the team and cost.
- *Q: Vertical vs. horizontal scaling?* — Vertical adds CPU/RAM to one server (simple, has a ceiling); horizontal adds nodes and splits data (near-unlimited, but brings partitioning, rebalancing and consistency problems).

**Your experience hook:** You have run PostgreSQL, Oracle, MongoDB and Redis side by side in production, so you can talk about polyglot persistence from real choices rather than theory.

**Beyond IBM:** In practice "one database per bounded context" plus CDC or events to move data between them is the common pattern; the hard part is keeping the copies consistent, which is where idempotent consumers and replayable logs come in.

---

## Relational databases
*Source: [IBM — What is a Relational Database?](https://www.ibm.com/think/topics/relational-databases)*

**In one line:** A relational database stores data in tables of rows and columns and links tables through primary and foreign keys, so related facts can be joined at query time instead of being duplicated.

**Key points**
- Based on Edgar F. Codd's 1970 relational model; IBM's System R project proved it and became the testing ground for SQL.
- Tables are joined on shared keys (e.g. `customer_id`), which supports normalisation: each fact is stored once.
- Relational systems are transactional and follow **ACID**: atomicity (all or nothing), consistency (constraints hold before and after), isolation (concurrent transactions do not see each other's half-done work) and durability (committed data survives crashes).
- An RDBMS (PostgreSQL, MySQL, Db2, Oracle) adds multi-user access, privileges, network access and structure on top of the model.
- Benefits listed: powerful joins and aggregation in SQL, indexes for fast lookup, reduced redundancy, stored procedures for reuse, and easy backup, replicas and disaster recovery.
- The article contrasts this with NoSQL families (key-value, document, wide-column, graph) and mentions the CAP trade-off.

**Compare / don't confuse with:** "C" in ACID (database constraints and invariants are respected) is not the "C" in CAP (every node returns the latest write).

**Interview angle**
- *Q: Explain ACID with a bank transfer.* — Debit and credit run in one transaction: either both commit or neither does (A); balances never break a `CHECK (balance >= 0)` rule (C); a concurrent reader never sees money "in flight" (I); once committed, a crash does not lose it because it is in the write-ahead log (D).
- *Q: What isolation anomalies do you know?* — Dirty reads, non-repeatable reads, phantom reads and write skew. Higher isolation levels (Repeatable Read, Serializable) prevent more of them at the cost of more conflicts or retries.

**Your experience hook:** On the MICHU lending platform, loan and repayment records across 6 banks are a textbook case for ACID transactions in PostgreSQL.

**Beyond IBM:** IBM says NoSQL databases "prioritize availability over consistency", which is too broad: MongoDB and HBase are usually described as CP, and many NoSQL engines now offer transactions. Also know normal forms (1NF to 3NF) and when you deliberately denormalise for reads.

---

## Structured Query Language (SQL)
*Source: [IBM — What Is Structured Query Language (SQL)?](https://www.ibm.com/think/topics/structured-query-language)*

**In one line:** SQL is the standard declarative language for defining, querying, changing and securing data in relational databases: you state what result you want and the engine decides how to get it.

**Key points**
- Created at IBM in the 1970s by Chamberlin and Boyce (first called SEQUEL); standardised by ANSI (1986) and ISO (1987). Vendors add dialects such as T-SQL and PL/SQL.
- A query passes through a **parser** (syntax, semantics, permissions), a **query optimizer** (picks an execution plan) and a **storage engine** (reads and writes the physical data).
- Building blocks: tables, constraints, data types, indexes, views, stored procedures, transactions, permissions.
- Command groups: **DDL** (CREATE, ALTER, DROP), **DML** (INSERT, UPDATE, DELETE), **DQL** (SELECT), **DCL** (GRANT, REVOKE), **TCL** (COMMIT, ROLLBACK).
- SQL vs. NoSQL: SQL databases are traditionally scaled vertically, use fixed schemas and ACID; NoSQL scales out, has flexible models and often follows BASE (basically available, soft state, eventually consistent).
- SQL injection remains a real threat when applications build queries from untrusted input.

**Compare / don't confuse with:** Declarative SQL vs. imperative code (Python, Java) that spells out each step.

**Interview angle**
- *Q: What is the logical order in which a SELECT is evaluated?* — FROM/JOIN, WHERE, GROUP BY, HAVING, SELECT (including window functions), DISTINCT, ORDER BY, LIMIT. That is why you cannot use a SELECT alias in WHERE but can in ORDER BY.
- *Q: How do you prevent SQL injection?* — Always use parameterised queries or an ORM's bound parameters, never string concatenation; add least-privilege database roles as a second layer.
- *Q: WHERE vs. HAVING?* — WHERE filters rows before grouping; HAVING filters groups after aggregation.

**Beyond IBM:** Interviewers for data roles expect fluency with window functions (`ROW_NUMBER`, `LAG`, running totals), CTEs, and `LEFT JOIN` pitfalls (a filter on the right table in WHERE silently turns it into an inner join).

---

## Primary key
*Source: [IBM — What Is a Primary Key?](https://www.ibm.com/think/topics/primary-key)*

**In one line:** A primary key is the column or set of columns whose values uniquely identify every row in a table; it can never be null or duplicated.

**Key points**
- Rules: unique, not null, one per table. It may be a single column or a **composite** of several.
- **Natural key**: a real-world attribute already in the data (employee ID, licence number). **Surrogate key**: a generated value with no business meaning (auto-increment, sequence, UUID).
- Integers are often recommended because they compare and index cheaply.
- Related keys: **unique key** (unique, may allow nulls), **candidate key** (any column set that could be the primary key), **super key** (any unique set, possibly with extra columns), **foreign key** (points to a primary key in another table).
- **Referential integrity** constraints stop orphan rows, e.g. an employee in a department that does not exist.
- Most RDBMSs create a unique index for the primary key automatically; keys are set with CREATE TABLE or ALTER TABLE.

**Compare / don't confuse with:** Primary key (identity of a row) vs. foreign key (a reference to another table's identity) vs. unique constraint (uniqueness only).

**Interview angle**
- *Q: Natural or surrogate key?* — Prefer a surrogate primary key and keep the natural key as a UNIQUE constraint. Natural keys change, get reused or collide (phone numbers, emails), and changing a primary key ripples through every foreign key.
- *Q: UUID or bigint?* — Bigint sequences are compact and index-friendly but leak ordering and are hard to merge across systems. UUIDs are globally unique, which helps merges and distributed inserts, but random UUIDv4 values scatter B-tree inserts; time-ordered UUIDv7 or ULID reduce that cost.

**Your experience hook:** You introduced UUID surrogate keys on the KYC/KYB platform to fix ID and phone-number collisions when consolidating 2M+ records from several source databases.

**Beyond IBM:** IBM's composite-key example (last name plus date of birth) would fail in practice because two people can share both; use it as a warning, not a model.

---

## NoSQL databases
*Source: [IBM — What Is a NoSQL Database?](https://www.ibm.com/think/topics/nosql-databases)*

**In one line:** NoSQL ("not only SQL") databases store data in non-tabular models such as documents, key-value pairs, wide columns or graphs, usually trading some relational features for flexible schemas and horizontal scale.

**Key points**
- Types: **key-value** (Redis, Memcached; caching, sessions), **document** (MongoDB, CouchDB; JSON/BSON records), **wide-column** (Cassandra, HBase; huge, sparse, write-heavy data), **graph** (Neo4j; relationships, fraud, recommendations), and **in-memory** stores.
- Built to be distributed: data is replicated and sharded across servers, so the system keeps running when nodes fail.
- Advantages listed: cost (commodity hardware, scale-out), flexibility for fast-changing data, replication, and speed.
- Use cases: relationship-heavy data (graph), low-latency apps such as gaming and ad tech (in-memory), traffic spikes in e-commerce (key-value).
- Microservices encourage **polyglot persistence**: each service picks the database that fits it.
- Relational and NoSQL often coexist in the same application.

**Compare / don't confuse with:** ACID (strong guarantees per transaction) vs. BASE (basically available, soft state, eventually consistent).

**Interview angle**
- *Q: When would you NOT pick NoSQL?* — When you need multi-entity invariants, ad hoc queries and joins across many entities, or strong consistency for money; a relational database with good indexes usually goes further than people expect.
- *Q: How do you model data in a NoSQL store?* — Model around access patterns: decide which queries must be fast, then embed or denormalise so each query hits one partition or document, accepting duplicated data and the work to keep it in sync.

**Your experience hook:** Your credit-scoring work runs on MongoDB aggregation pipelines, while transactional lending data lives in PostgreSQL: a real polyglot setup.

**Beyond IBM:** "Schemaless" really means schema-on-read: the schema lives in application code instead. Also, relational databases are no longer only vertical: read replicas, partitioning and distributed SQL (Citus, CockroachDB, Spanner-style systems) blur the line.

---

## Online transaction processing (OLTP)
*Source: [IBM — What Is Online Transactional Processing (OLTP)?](https://www.ibm.com/think/topics/oltp)*

**In one line:** OLTP systems run large numbers of small, concurrent, atomic transactions in real time, such as payments, bookings and balance checks.

**Key points**
- Workload: many simple inserts, updates, deletes and point lookups, each touching a few rows.
- Every transaction is atomic: it fully succeeds or is rolled back, never left half-done.
- Concurrency control stops two users changing the same data at once (no double-booked rooms, no overdraft on a joint account).
- Success is measured in transactions per second and millisecond response times; data is indexed for fast lookup.
- Must be available around the clock, with frequent and incremental backups.
- OLTP systems are usually the source of data for OLAP (analytical) systems.

**Compare / don't confuse with:** OLTP (many short read/write transactions on current data, normalised, row-oriented) vs. OLAP (few heavy read-only queries over large historical data, denormalised, column-oriented).

**Interview angle**
- *Q: Why not run analytics directly on the OLTP database?* — Long scans and aggregations compete for CPU, I/O and locks with customer transactions and hurt latency. Offload to a read replica, or better, move data out with CDC or ETL into a warehouse built for scans.
- *Q: How do you prevent double booking?* — Use a transaction with a row lock (`SELECT ... FOR UPDATE`), a unique constraint on the booked slot, or optimistic concurrency with a version column; plus idempotency keys for client retries.

**Your experience hook:** MICHU is an OLTP system (2M+ users, ~900 req/s on PostgreSQL), and your Airflow + Redpanda pipelines are exactly the OLTP-to-analytics hand-off.

**Beyond IBM:** IBM describes OLAP as running on "multidimensional databases"; today it mostly means columnar MPP warehouses or lakehouse engines (BigQuery, Snowflake, Spark/Trino on Parquet). HTAP systems try to serve both from one engine.

---

## Vector database
*Source: [IBM — What Is a Vector Database?](https://www.ibm.com/think/topics/vector-database)*

**In one line:** A vector database stores high-dimensional embeddings and indexes them for fast similarity search, so you can find items that are close in meaning rather than identical in wording.

**Key points**
- **Embeddings** are numeric vectors produced by a model; similar items (e.g. "car" and "vehicle") land close together in vector space.
- Three core functions: **storage** (vectors plus metadata such as source, date, category), **indexing** and **search**.
- Indexes support **approximate nearest neighbour (ANN)** search: **HNSW** (a layered graph) and **LSH** (hash similar vectors into the same bucket); **product quantization** compresses vectors to save memory.
- Similarity measures: cosine, dot product, Euclidean, Jaccard, Hamming.
- Hybrid search combines vector similarity with metadata filters (date range, category).
- Use cases: RAG, chatbots, recommendations, anomaly detection. Not ideal for summarising a whole corpus, where the model needs all the context rather than the nearest few chunks.
- Options: standalone (Pinecone), open source (Milvus, Weaviate), extensions (PostgreSQL **pgvector**), search engines with vector support (OpenSearch), lakehouse integrations, serverless offerings.

**Compare / don't confuse with:** Keyword search (exact token match, e.g. BM25) vs. semantic vector search (meaning match).

**Interview angle**
- *Q: Why ANN instead of exact search?* — Exact k-NN compares the query to every vector, which is too slow at millions of items; ANN trades a little recall for big speed gains, and you tune that trade-off (e.g. HNSW `M` and `ef_search`).
- *Q: Do you need a dedicated vector database?* — Often not at first: pgvector keeps vectors next to relational data with transactions and joins. A dedicated engine pays off at very large scale or high query rates.

**Your experience hook:** You build customer-facing RAG systems, so you can discuss embedding storage, metadata filtering and retrieval quality from production experience.

**Beyond IBM:** When you change the embedding model, every stored vector must be regenerated, because vectors from different models are not comparable; version embeddings like any other derived dataset.

---

## Vector databases for RAG
*Source: [IBM — Vector Databases for RAG](https://www.ibm.com/think/topics/rag-vector-database)*

**In one line:** In retrieval-augmented generation, a vector database is the retrieval layer that finds relevant chunks of trusted data and feeds them into the LLM's prompt so answers are grounded and current.

**Key points**
- RAG fixes LLM weaknesses (knowledge cutoff, hallucination, missing domain knowledge) by adding "non-parametric memory" that is queried at run time.
- Flow: user prompt, embed the query, retrieve similar embeddings, rank by relevance, augment the prompt with the retrieved chunks, generate the answer.
- Components: knowledge base, embedding model, vector database, retriever, integration (orchestration and prompt) layer, generator (the LLM).
- RAG vs. fine-tuning: fine-tuning changes what the model knows; RAG changes what it can access, and you update data instead of retraining.
- Design concerns: retrieval quality, **chunking** (too large loses precision, too small loses context), context-window limits, added latency, and security, since embeddings can leak information about the source text.
- Trends: agentic (multi-step) retrieval, hybrid retrieval (vector + keyword + metadata + graph), real-time ingestion, multimodal RAG.
- Vector search is common but not required; keyword or structured retrieval also works.

**Compare / don't confuse with:** RAG (retrieve at query time, model unchanged) vs. fine-tuning (change model weights).

**Interview angle**
- *Q: Your RAG bot gives wrong answers. How do you debug it?* — Split retrieval from generation: first check whether the right chunks were retrieved (recall@k on a labelled question set). If not, fix chunking, the embedding model, hybrid search or reranking. If they were, fix the prompt or the model's grounding.
- *Q: How do you stop one customer seeing another's documents?* — Enforce access control at retrieval time with metadata filters (tenant ID, permissions) applied inside the query, not after generation.

**Your experience hook:** Your customer-facing RAG systems face exactly these issues, and your PII pseudonymisation and access-control work applies directly to securing a retrieval pipeline.

**Beyond IBM:** A common production recipe is hybrid retrieval (BM25 + vectors, merged with reciprocal rank fusion) followed by a cross-encoder reranker on the top candidates.

---

## Database as a service (DBaaS)
*Source: [IBM — What Is Database as a Service (DBaaS)?](https://www.ibm.com/think/topics/dbaas)*

**In one line:** DBaaS is a cloud service where the provider runs the database for you (hardware, installation, patching, backups, high availability), and you pay for what you use.

**Key points**
- Benefits: no upfront hardware, scale up or down on demand, less admin work, fast self-service provisioning, built-in encryption and access control, uptime SLAs.
- Choosing one starts with choosing the right database technology, then checking the provider supports it.
- **Primary data stores** (PostgreSQL, MySQL, MongoDB) are general-purpose with flexible models and strong integrity. **Auxiliary data stores** (Redis, etcd, Elasticsearch, JanusGraph) excel at one job; use them only when that job fits.
- Distributed engines scale further but are harder to run and may give up features or performance.
- Prototype and load-test with real traffic before committing.
- Compare the whole platform: management tools, logging, key management, monitoring.

**Compare / don't confuse with:** Self-managed database on cloud VMs (you run everything) vs. DBaaS (provider runs the platform; you still own data design).

**Interview angle**
- *Q: What does a managed database NOT do for you?* — It does not design your schema, add indexes, tune queries, size connection pools or test your restore procedure. Shared responsibility still leaves performance, data modelling and access policies with you.
- *Q: Downsides of DBaaS?* — Cost at steady high load, limited superuser access and extensions, version upgrade timing set by the provider, and lock-in to provider-specific features.

**Beyond IBM:** Always run a restore drill: a backup you have never restored is not a backup, managed or not.

---

## CAP theorem
*Source: [IBM — What Is the CAP Theorem?](https://www.ibm.com/think/topics/cap-theorem)*

**In one line:** The CAP theorem says that when a network partition splits a distributed system, it must choose between consistency (every read sees the latest write) and availability (every request gets a response).

**Key points**
- **C**onsistency: all clients see the same data at the same time, whichever node they ask. **A**vailability: every working node answers every request. **P**artition tolerance: the system keeps working when nodes cannot talk to each other.
- Proposed by Eric Brewer (2000), proved by Gilbert and Lynch (2002).
- **CP** systems stop serving (part of) the data during a partition to stay consistent. **AP** systems keep answering but may return stale data, then reconcile later. **CA** is only possible without partitions, so it is not a real option for distributed systems; a single-node or replicated PostgreSQL is often described this way.
- **MongoDB** is CP: one primary per replica set takes all writes; if it fails, secondaries elect a new primary and writes pause until then.
- **Cassandra** is AP: masterless, any node accepts writes, eventual consistency with repair.
- Microservices: choose AP stores (Cassandra, CouchDB) where staleness is acceptable, and strongly consistent stores (PostgreSQL) for payments and orders.

**Compare / don't confuse with:** CAP consistency (linearizable reads across nodes) vs. ACID consistency (constraints hold within one database).

**Interview angle**
- *Q: Is MongoDB CP or AP?* — By default CP-leaning (single primary, majority elections), but it is tunable: reading from secondaries or using weak write concerns moves it toward AP behaviour. Use `writeConcern: majority` and `readConcern: majority` for stronger guarantees.
- *Q: How does Cassandra tune consistency?* — Per query, with consistency levels; if reads plus writes acknowledged exceed the replication factor (R + W > RF, e.g. QUORUM/QUORUM), reads see the latest write.

**Your experience hook:** Your MongoDB to Oracle change-stream ETL reads the replica set's oplog, so primary elections and resume tokens are part of keeping that pipeline correct.

**Beyond IBM:** CAP only describes behaviour during a partition. PACELC adds the everyday trade-off: Else (no partition), choose between Latency and Consistency. That is the more useful framing in design interviews.

---

## Database schema
*Source: [IBM — What Is a Database Schema?](https://www.ibm.com/think/topics/database-schema)*

**In one line:** A schema is the blueprint of a database: the tables, fields, data types, constraints and relationships that define how data is organised, without containing the data itself.

**Key points**
- Schema design is data modelling; diagrams document it for DBAs, developers and analysts.
- **Schema vs. instance**: the schema is the stable structure; an instance is the data at one moment and changes constantly.
- Three levels: **conceptual** (business view, entities and rules), **logical** (tables, fields, relationships, constraints, no tech details), **physical** (actual types, storage and DDL for a specific engine).
- **Star schema**: one central fact table surrounded by denormalised dimension tables; simpler and faster to query. **Snowflake schema**: dimensions are normalised into sub-dimensions; less redundancy, more joins, slower queries.
- Benefits: access control (e.g. isolating PII in its own schema), shared documentation, data integrity and normalisation.

**Compare / don't confuse with:** Schema-on-write (relational: validate on insert) vs. schema-on-read (data lakes, document stores: interpret on query).

**Interview angle**
- *Q: How do you change a schema on a live system without downtime?* — Use expand-and-contract: add the new column or table (backward compatible), deploy code that writes both, backfill, switch reads, then drop the old structure in a later release. Run migrations through a versioned tool (Alembic, Flyway, Django migrations).
- *Q: Star or snowflake for a warehouse?* — Usually star: fewer joins and simpler for BI users; snowflake only where a dimension is very large or shared.

**Your experience hook:** Your MongoDB to Oracle ETL had to map flexible documents onto a fixed relational schema, which is the schema-on-read vs. schema-on-write gap in practice.

**Beyond IBM:** In PostgreSQL, "schema" also means a namespace inside a database (`public`, `staging`), a handy unit for permissions and multi-tenant layouts.

---

## MongoDB
*Source: [IBM — What Is MongoDB?](https://www.ibm.com/think/topics/mongodb)*

**In one line:** MongoDB is an open source document database that stores flexible BSON (binary JSON) documents in collections and scales out through replica sets and sharding.

**Key points**
- Documents in a collection need not share the same fields (dynamic schema); data is queried with MQL and the aggregation pipeline.
- **Sharding** spreads data across machines for horizontal scale; secondary and geospatial indexes are supported.
- Strong ad hoc query support and drivers for most languages (Python, Node.js, Java and others).
- Use cases: mobile back ends, real-time analytics, content management, and integration with Hadoop for large-scale processing.
- IBM's comparison: MySQL is better when integrity and isolation are critical; MongoDB when flexibility and speed matter. Cassandra offers SQL-like syntax and easy replication but less flexible querying.

**Compare / don't confuse with:** Embedding (store related data inside one document; one read, atomic per document) vs. referencing (store IDs and look up; avoids duplication and unbounded document growth).

**Interview angle**
- *Q: Does MongoDB support transactions?* — Single-document writes have always been atomic; multi-document ACID transactions exist since 4.0 on replica sets and 4.2 on sharded clusters, but they cost performance, so design documents so most operations touch one document.
- *Q: How do you design a compound index?* — Follow the ESR rule: Equality fields first, then Sort fields, then Range fields; check with `explain()` that the query uses an index scan (IXSCAN) rather than a collection scan (COLLSCAN).
- *Q: What makes a good shard key?* — High cardinality, even write distribution, and present in most queries so they route to one shard; a monotonically increasing key (e.g. timestamp) creates a hot shard.

**Your experience hook:** You run credit scoring on MongoDB aggregation pipelines and built incremental MongoDB to Oracle ETL with change streams covering inserts, updates and deletes.

**Beyond IBM:** IBM's claim that MongoDB has "higher performance" than MySQL is marketing; performance depends entirely on the workload and data model.

---

## PostgreSQL
*Source: [IBM — What Is PostgreSQL?](https://www.ibm.com/think/topics/postgresql)*

**In one line:** PostgreSQL is a mature, standards-compliant open source relational database known for reliability, extensibility and support for both relational and semi-structured (JSON) data.

**Key points**
- Descends from Michael Stonebraker's POSTGRES project at Berkeley (1986); SQL support came in the mid-1990s.
- **MVCC** (multiversion concurrency control): readers do not block writers and writers do not block readers, because each transaction sees a snapshot.
- **Write-ahead log (WAL)** records every change; it enables crash recovery, replication and **point-in-time recovery** (tools such as pgBackRest; managed services automate it).
- High availability through synchronous or asynchronous replication to standby servers.
- Rich extension ecosystem: **PostGIS** for geospatial, time-series extensions, pgvector; stored procedures in several languages.
- Use cases: general OLTP (e-commerce, CRM, ledgers), moderate analytics, GIS, consolidating legacy databases without licence costs.

**Compare / don't confuse with:** Read replicas (scale reads, possible replication lag) vs. partitioning (split a big table) vs. sharding (split data across servers, e.g. Citus).

**Interview angle**
- *Q: What is the cost of MVCC?* — Updates and deletes leave dead row versions, so `VACUUM` (autovacuum) must clean them and refresh visibility info; neglected vacuum causes table bloat and, in the extreme, transaction ID wraparound.
- *Q: Which index types do you use?* — B-tree for equality and range (default), GIN for `jsonb`, arrays and full-text, GiST for geometric/range types, BRIN for huge naturally ordered tables; plus partial indexes (`WHERE status = 'active'`) and covering indexes (`INCLUDE`).
- *Q: Default isolation level?* — Read Committed. Repeatable Read gives snapshot isolation; Serializable adds conflict detection, so the app must retry on serialisation failures.

**Your experience hook:** You tuned PostgreSQL to serve about 900 requests per second on MICHU, so be ready to walk through what you changed (indexes, query plans, connection pooling, configuration).

**Beyond IBM:** At high request rates, connections are often the bottleneck before CPU: each PostgreSQL connection is a process, so a pooler such as PgBouncer is standard.

---

## Redis
*Source: [IBM — What is Redis Explained?](https://www.ibm.com/think/topics/redis)*

**In one line:** Redis is an open source, in-memory key-value data-structure server used mainly as a cache, session store, queue and fast counter store.

**Key points**
- Keeps all data in RAM, giving sub-millisecond reads and writes; replication can place copies close to users.
- Supports rich data types: strings, hashes, lists, sets, sorted sets, bitmaps, HyperLogLogs, geospatial indexes (and streams).
- **Persistence**: periodic snapshots plus an append-only log of changes, so data can survive restarts.
- **Redis Sentinel** monitors primaries and replicas and fails over automatically; **Redis Cluster** shards data across nodes.
- **Pub/Sub** for messaging; lists for task queues; Lua scripting for atomic multi-step operations.
- Use cases: database caching, real-time analytics, leaderboards, location-based features. Often paired with an on-disk store such as MongoDB.
- vs. Memcached: Memcached is a simpler cache of opaque values; Redis offers structures, persistence and replication.

**Compare / don't confuse with:** Redis Pub/Sub (fire-and-forget; offline subscribers miss messages) vs. Redis Streams or a list-based reliable queue (messages persist and can be acknowledged).

**Interview angle**
- *Q: Explain cache-aside and its risks.* — Read from cache; on a miss, read the database and populate the cache with a TTL; on write, update the database and delete the cache key. Risks: stale data, thundering herd on popular keys (use locking or jittered TTLs), and treating the cache as a source of truth.
- *Q: What happens when Redis memory is full?* — It follows the `maxmemory-policy`: evict (e.g. `allkeys-lru`) or reject writes (`noeviction`). Queues need `noeviction`, caches usually LRU or LFU.
- *Q: RDB vs. AOF?* — RDB snapshots are compact and fast to restore but lose data since the last snapshot; AOF logs every write, losing at most about a second with `everysec` fsync.

**Your experience hook:** You built RabbitMQ and Redis queues with idempotency, retries and dead-letter queues on the KYC/KYB platform.

**Beyond IBM:** IBM oddly calls PostgreSQL a NoSQL database here; ignore that. Note also the 2024 licence change, after which the Linux Foundation fork Valkey appeared; managed clouds now offer both. Redis 8 (2025) added AGPL as an open source licence option again.

---

## Elasticsearch
*Source: [IBM — What Is Elasticsearch?](https://www.ibm.com/think/topics/elasticsearch)*

**In one line:** Elasticsearch is a distributed search and analytics engine built on Apache Lucene that indexes JSON documents for fast full-text search and aggregations over a REST API.

**Key points**
- Released by Elastic in 2010; written in Java on the **Lucene** library.
- Lucene builds keyword-centred (inverted) indexes, so a term lookup finds matching documents without scanning them.
- Scales horizontally across nodes and supports cross-cluster replication per index.
- Features: autocomplete, many client languages, a large plugin and integration ecosystem.
- Use cases: application performance monitoring (correlating logs and metrics), SIEM (searching billions of security log lines in near real time) and enterprise search across many systems.

**Compare / don't confuse with:** A search engine (relevance ranking, text analysis, near-real-time visibility) vs. a primary database (transactions, strong consistency, source of truth).

**Interview angle**
- *Q: What is an inverted index?* — A map from each term to the list of documents (and positions) containing it, built after text is tokenised and normalised by analysers; it makes keyword search fast.
- *Q: Would you use Elasticsearch as your main database?* — Usually not: it has no multi-document transactions and new documents become searchable only after a refresh (about one second by default). Keep the source of truth in a database and feed Elasticsearch via CDC or events, so you can rebuild indexes.

**Your experience hook:** ELK is part of your toolset for logs and monitoring, which is the APM/observability use case IBM describes.

**Beyond IBM:** The number of primary shards is fixed when an index is created, so plan sizes up front or use time-based indexes with rollover; uncontrolled dynamic mappings can cause "mapping explosion".

---

## OpenSearch
*Source: [IBM — What Is OpenSearch?](https://www.ibm.com/think/topics/opensearch)*

**In one line:** OpenSearch is the Apache 2.0-licensed fork of Elasticsearch and Kibana, used for search, log analytics, observability and security analytics.

**Key points**
- Created by AWS in 2021 from the last Apache 2.0 versions after Elastic moved to SSPL and the Elastic License; now community-governed and Apache 2.0.
- Keeps much API and query compatibility with older Elasticsearch, easing migration.
- Architecture: a **cluster** of **nodes** (cluster-manager/master, data, coordinating). An **index** is a logical collection of JSON **documents**, split into **primary shards** and **replica shards**; each shard is a standalone Lucene index.
- Querying: a coordinating node fans the query out to relevant shards and merges results (scatter-gather).
- Features: full-text search, aggregations, SQL and PPL query languages, Index State Management (rollover, retention), Data Prepper ingestion, fine-grained access control, OpenSearch Dashboards.
- Plugins: anomaly detection, k-NN vector search, ML Commons, cross-cluster replication, trace analytics.

**Compare / don't confuse with:** OpenSearch vs. Elasticsearch: shared origin, now separate roadmaps; choose on licence, governance and features, not just API.

**Interview angle**
- *Q: What do replica shards give you?* — Fault tolerance (a replica is promoted if a primary's node dies) and more read throughput, since searches can be served by any copy; they cost extra storage and indexing work.
- *Q: How do you manage ever-growing log indexes?* — Use time-based indexes with rollover and an ISM policy to move old indexes to cheaper storage and delete them after the retention period.

**Your experience hook:** Your ELK experience transfers directly, since OpenSearch Dashboards descends from Kibana.

**Beyond IBM:** Elastic later added AGPL as an open source licence option for Elasticsearch (2024), so "Elasticsearch is not open source" is no longer accurate; the licence difference is now about which open source licence.

---

## etcd
*Source: [IBM — What Is etcd?](https://www.ibm.com/think/topics/etcd)*

**In one line:** etcd is a small, strongly consistent, distributed key-value store for critical configuration and state, best known as the database behind Kubernetes.

**Key points**
- Kubernetes stores all cluster state in etcd and uses etcd's **watch** feature to react when desired and actual state differ.
- Qualities: fully replicated, highly available, consistent (reads return the latest write), simple HTTP/JSON access, TLS security. Disk speed matters, so use SSDs.
- Uses the **Raft** consensus algorithm: an elected leader appends each write to its log, commits once a majority of members have it, and a new election runs if followers stop hearing from the leader.
- Created by the CoreOS team; donated to the CNCF in 2018.
- vs. ZooKeeper: etcd adds dynamic membership changes, MVCC, reliable watches and broader language support. vs. Consul: similar Raft store, but Consul is a wider service-networking tool.
- vs. Redis: Redis is faster with richer types; etcd persists everything to disk and favours consistency and fault tolerance, so it suits configuration, not caching.

**Compare / don't confuse with:** Consensus stores (etcd, ZooKeeper: small, CP, coordination) vs. general databases (large data volumes, rich queries).

**Interview angle**
- *Q: Why do etcd clusters have 3 or 5 members?* — Raft needs a majority to commit; with 2f+1 members it survives f failures. Three tolerate one failure, five tolerate two; an even number adds cost without adding tolerance.
- *Q: Where is etcd on CAP?* — CP: a minority partition cannot elect a leader or accept writes.

**Beyond IBM:** etcd is meant for small data (a few GB at most); use it for leader election, locks and config, not as an application database.

---

## Apache Cassandra
*Source: [IBM — What Is Cassandra?](https://www.ibm.com/think/topics/cassandra)*

**In one line:** Cassandra is an open source, masterless, wide-column NoSQL database designed for very high write throughput and always-on availability across multiple data centres.

**Key points**
- Created at Facebook in 2007, combining ideas from Amazon's Dynamo (partitioning, replication) and Google's Bigtable (data model); Apache project since 2009.
- **Peer-to-peer**: every node is equal, so there is no single point of failure; data is spread by **partition key** and copied according to the **replication factor**.
- Write path: commit log, then in-memory **memtable**, then immutable **SSTables** on disk, merged by **compaction** (an LSM-tree design), which makes writes fast.
- **Tunable consistency** per query lets you trade availability against consistency; **linear scalability** by adding nodes.
- **CQL** looks like SQL but works on denormalised, partition-aligned tables, without joins and with limited ad hoc querying.
- Use cases: IoT sensor data, e-commerce carts, streaming platforms, messaging, time series.

**Compare / don't confuse with:** Cassandra (AP, masterless, write-heavy, query-first modelling) vs. MongoDB (CP-leaning, single primary per shard, flexible querying).

**Interview angle**
- *Q: How do you model data in Cassandra?* — Query-first: one table per query pattern, partition key chosen so each query reads one partition, clustering columns to sort within it, and partitions bounded in size (e.g. bucket time series by day).
- *Q: Main pitfalls?* — Hot or unbounded partitions, heavy deletes creating tombstones that slow reads, and expecting joins or ad hoc filters (`ALLOW FILTERING` is a warning sign).

**Beyond IBM:** Cassandra offers lightweight transactions (compare-and-set using Paxos) for rare cases like uniqueness checks, but they are much slower than normal writes.

---

## Apache HBase
*Source: [IBM — What Is HBase?](https://www.ibm.com/think/topics/hbase)*

**In one line:** HBase is a column-family NoSQL database that runs on top of HDFS and gives random, real-time read/write access to very large, sparse tables.

**Key points**
- Not relational and has no native SQL; applications use the Java API (plus REST, Thrift, Avro). Hive or other engines can add SQL on top.
- Tables have rows and columns, and every access goes through the **row key** (its primary key).
- Columns are grouped into **column families** that are stored together; families are defined up front, but new columns can be added at any time.
- Architecture: a **master** manages the cluster; **region servers** hold slices (regions) of tables. **ZooKeeper** handles coordination; production setups should run a dedicated ZooKeeper ensemble.
- Fault tolerant through HDFS replication, but sensitive to master availability.
- Example: a table of server logs keyed by record, with columns such as timestamp and server name.

**Compare / don't confuse with:** HBase (single master, strongly consistent per row, Hadoop-based) vs. Cassandra (masterless, tunable consistency, self-contained).

**Interview angle**
- *Q: Why is row-key design critical in HBase?* — Rows are stored sorted by key and split into regions, so sequential keys (timestamps) send all writes to one region server; salting, hashing or reversing keys spreads the load while keeping useful scans possible.

**Beyond IBM:** HBase follows Google's Bigtable design. New projects rarely choose it today outside existing Hadoop estates; cloud equivalents (Bigtable) or Cassandra-style stores are more common.

---

## Apache CouchDB
*Source: [IBM — What Is CouchDB?](https://www.ibm.com/think/topics/couchdb)*

**In one line:** CouchDB is an open source JSON document database built around an HTTP API and multi-master replication, which makes it suited to offline-first and sync-heavy apps.

**Key points**
- Schema-free JSON documents with binary attachments; written in Erlang; Apache project since 2008.
- **Bi-directional replication**: any copy can sync with any other, across servers, devices and browsers (via the compatible PouchDB).
- **Offline first**: apps write locally and sync when back online.
- **Views** built with MapReduce in design documents give precomputed indexes and aggregates that are fast to read.
- **REST/HTTP API** for full CRUD, so it sits behind standard load balancers and works with any HTTP client.
- **MVCC with no read locks**: readers see the version that existed when they started; a conflicting concurrent write is rejected.
- Scales with partitioning and replication.

**Compare / don't confuse with:** CouchDB (multi-master sync, AP, conflict handling) vs. MongoDB (single primary per replica set, CP-leaning, richer query language).

**Interview angle**
- *Q: What is the cost of multi-master replication?* — Conflicts: two devices can edit the same document offline. CouchDB keeps the losing revisions and picks a deterministic winner, but the application must detect and merge conflicts for correctness.

**Beyond IBM:** Besides MapReduce views, CouchDB has the Mango JSON query language with indexes, which is easier for simple lookups.

---

## Presto
*Source: [IBM — What Is Presto?](https://www.ibm.com/think/topics/presto)*

**In one line:** Presto is an open source, distributed SQL query engine that queries data where it lives (data lakes, databases, streams) without storing data itself.

**Key points**
- Built at Facebook to run interactive queries on a huge Hadoop warehouse; open-sourced in 2013.
- Separates compute from storage: **connectors** plug in sources such as HDFS/Hive, object storage, Cassandra, Kafka, MySQL, PostgreSQL, MongoDB, BigQuery.
- **MPP architecture**: a **coordinator** parses, plans and schedules queries; **workers** run tasks and exchange data in memory (streaming shuffle); a resource manager gives a cluster-wide view.
- In-memory pipelined execution avoids writing to disk between stages, unlike classic MapReduce-style SQL-on-Hadoop, so it is low latency.
- Standard ANSI SQL, including joins, subqueries and aggregations.
- Use cases: ad hoc exploration, federated BI reports across sources, querying the lake directly without heavy ETL, and preparing ML features.

**Compare / don't confuse with:** Query engine (Presto/Trino, Spark SQL: compute only) vs. database or warehouse (stores and manages the data too).

**Interview angle**
- *Q: When would you use Presto/Trino rather than Spark?* — For interactive, short-to-medium SQL queries and federation across sources; Spark is better for long batch ETL, complex transformations and jobs that need fault-tolerant retries mid-query.
- *Q: What makes lake queries fast?* — Columnar formats (Parquet, ORC), partition pruning, predicate and projection pushdown, file sizes that are neither tiny nor huge, and table statistics.

**Beyond IBM:** The 2019 fork IBM mentions (PrestoSQL) was renamed **Trino** in 2020 and is now the more widely used branch; AWS Athena is built on Presto/Trino. Today such engines usually query open table formats like Apache Iceberg.

---

## Query optimization
*Source: [IBM — What Is Query Optimization?](https://www.ibm.com/think/topics/query-optimization)*

**In one line:** Query optimization is the database's process of choosing the cheapest way to execute a query from many equivalent plans, guided by statistics and a cost model.

**Key points**
- Approaches: rule-based (older, fixed rules), **cost-based** (modern: estimate CPU, I/O, memory and network cost for each plan) and heuristics to limit the search.
- Inputs: **statistics** (row counts, value distributions, index selectivity) and **cardinality estimates** (how many rows each step returns); stale stats lead to bad plans.
- **Access paths**: full table scan, index scan, index seek, index-only scan.
- **Join algorithms**: nested loop (small or indexed inner side), hash join (large unsorted inputs), merge join (both inputs sorted).
- Steps: parse and validate, rewrite (predicate pushdown, subquery flattening, join reordering, removing redundant sorts), generate plans, estimate cost, choose the plan; inspect it with `EXPLAIN`.
- Challenges: inaccurate statistics, data skew, very complex queries, changing data.
- Manual techniques: targeted indexes (balancing write cost), fresh statistics, early filtering, fewer joins or careful denormalisation, selecting only needed columns, partitioning and caching.

**Compare / don't confuse with:** Query optimization (automatic, done by the engine) vs. query tuning (manual: rewriting queries, adding indexes, changing settings).

**Interview angle**
- *Q: A query became slow overnight. What do you do?* — Run `EXPLAIN ANALYZE` and compare estimated vs. actual rows; a big gap points to stale statistics (run `ANALYZE`) or skew. Then check for a missing or unused index, a changed parameter value, lock waits, or a plan change after data growth.
- *Q: Why might an index not be used?* — The predicate is not "sargable" (e.g. `WHERE YEAR(order_date) = 2022` or a function or cast on the column), low selectivity makes a scan cheaper, the column order in a composite index does not match the filter, or there is a type mismatch. Rewrite as a range: `order_date >= '2022-01-01' AND order_date < '2023-01-01'`.

**Your experience hook:** Tuning PostgreSQL to ~900 req/s, and using Splink blocking to cut 14.7B candidate pairs to 77M, are both about avoiding unnecessary work before the expensive comparison step.

**Beyond IBM:** Common application-side wins are removing N+1 query patterns, batching writes, and replacing large `OFFSET` pagination with keyset (seek) pagination.

---

## Entity relationship diagram (ERD)
*Source: [IBM — What is an Entity Relationship Diagram?](https://www.ibm.com/think/topics/entity-relationship-diagram)*

**In one line:** An ER diagram is a visual model of the entities in a system, their attributes and the relationships (with cardinality) between them, used to design and explain a database.

**Key points**
- Uses: database design and data modelling, scoping data integration, debugging existing schemas, and business process reengineering.
- **Entities** are things you store data about (people, events, concepts, objects). **Strong** entities stand alone; **weak** entities depend on an owner (order line items depend on an order). **Associative** entities resolve many-to-many relationships and become junction tables.
- **Attributes**: simple, composite (address), derived (computed, e.g. pay), multivalued; key attributes (super, candidate, primary, foreign).
- **Relationships** connect entities; participation can be total or partial.
- **Cardinality**: one-to-one, one-to-many, many-to-many.
- Model levels: conceptual (least detail), logical (attributes defined), physical (keys, types, cardinality; ready to build).
- Notation styles: Chen, crow's foot (most common in tools), Bachman, IDEF1X, Barker (Oracle).

**Compare / don't confuse with:** ERD (structure of data) vs. database schema (the implemented rules) vs. data flow diagram (how data moves between processes and stores).

**Interview angle**
- *Q: How do you implement a many-to-many relationship?* — With a junction table holding foreign keys to both sides (often a composite primary key of the two), plus any attributes of the relationship itself, such as enrolment date or grade.
- *Q: Walk me through modelling a lending system.* — Identify entities (customer, bank, loan, repayment), their keys and relationships (a customer has many loans; a loan has many repayments; a loan belongs to one bank), then refine to logical and physical models with types, constraints and indexes on the foreign keys.

**Your experience hook:** Consolidating 2M+ records from multiple source databases into one KYC/KYB model required reconciling different entity definitions and keys, which is ER modelling under real constraints.

**Beyond IBM:** IBM's description of Chen participation lines is reversed from the usual convention: in Chen notation a double line normally means total participation and a single line partial.

---

## Self-test

1. What does each letter of ACID guarantee, and how does ACID consistency differ from CAP consistency?
<details><summary>Answer</summary>Atomicity: all of a transaction's changes happen or none do. Consistency: constraints and invariants hold before and after. Isolation: concurrent transactions do not see each other's partial work. Durability: committed changes survive crashes (via the WAL). ACID consistency is about rules inside one database; CAP consistency means every node returns the latest write (linearizability) in a distributed system.</details>

2. Why would you choose a surrogate key over a natural key, and what is the trade-off between bigint and UUID keys?
<details><summary>Answer</summary>Natural keys can change, be reused or collide (phone numbers, emails), and changing a primary key cascades through foreign keys. Use a surrogate primary key and keep the natural key as a UNIQUE constraint. Bigints are small and index-friendly but hard to merge across systems; UUIDs are globally unique but random UUIDv4 values hurt B-tree locality, which UUIDv7/ULID reduce.</details>

3. During a network partition, what do CP and AP systems do? Give an example of each.
<details><summary>Answer</summary>CP systems refuse some requests (e.g. writes on the minority side) to avoid serving inconsistent data: MongoDB replica sets, etcd, HBase. AP systems keep answering on all sides and reconcile later, so reads may be stale: Cassandra, CouchDB. PACELC adds that, even without a partition, systems trade latency against consistency.</details>

4. In Cassandra with replication factor 3, which read and write consistency levels guarantee that reads see the latest write?
<details><summary>Answer</summary>Any combination where R + W > RF, e.g. QUORUM writes (2) and QUORUM reads (2): 2 + 2 > 3, so the read and write replica sets always overlap.</details>

5. What is MVCC, which databases in this section use it, and what maintenance does it require in PostgreSQL?
<details><summary>Answer</summary>Multiversion concurrency control keeps several versions of a row so readers see a consistent snapshot without blocking writers. PostgreSQL, CouchDB and etcd use it. In PostgreSQL, old row versions accumulate and must be removed by VACUUM/autovacuum to avoid bloat and transaction ID wraparound.</details>

6. Give three reasons a query might not use an existing index.
<details><summary>Answer</summary>A function or cast on the indexed column (non-sargable predicate, e.g. YEAR(date) = 2022); low selectivity so a full scan is cheaper; composite index column order that does not match the filter; stale statistics causing a bad estimate; or a type mismatch between column and parameter.</details>

7. Describe the cache-aside pattern with Redis and two of its risks.
<details><summary>Answer</summary>Read the cache; on a miss, read the database and store the result with a TTL; on writes, update the database and delete the cache key. Risks: stale reads between write and invalidation, thundering herd when a hot key expires, and memory pressure causing evictions (set an appropriate maxmemory-policy).</details>

8. What are the steps of a RAG request, and how would you tell whether a bad answer is a retrieval or a generation problem?
<details><summary>Answer</summary>Embed the query, retrieve similar chunks from the vector store (often with metadata filters), rank or rerank them, add them to the prompt, generate. Check whether the correct chunks appear in the top-k results: if not, it is retrieval (chunking, embeddings, hybrid search, reranking); if they do, it is the prompt or the model's grounding.</details>

9. When would you use Presto/Trino, and how is it different from a database?
<details><summary>Answer</summary>For interactive SQL over data in lakes and multiple sources (federated queries) without first moving it into a warehouse. It is compute only: it stores no data and reads through connectors, using an MPP coordinator/worker design with in-memory execution.</details>

10. How do you implement a many-to-many relationship from an ERD in a relational schema, and what is a weak entity?
<details><summary>Answer</summary>Create a junction (associative) table with foreign keys to both entities, usually a composite primary key, plus any attributes of the relationship. A weak entity cannot be identified on its own and depends on an owner entity, such as order line items that only exist within an order.</details>
