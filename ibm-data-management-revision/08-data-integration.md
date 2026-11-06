# 8. Data integration

> This section covers how data gets from the systems that produce it into a shape and place where other systems and people can use it: ETL, ELT, change data capture (CDC), real-time and cloud integration, AI-assisted integration, interoperability, data contracts and the main integration techniques. In data engineering interviews this is home ground. Expect to justify ETL vs. ELT, design an incremental pipeline, explain how CDC handles deletes, ordering and restarts, and say how you stop upstream schema changes from breaking downstream consumers.

## Data integration
*Source: [IBM — What Is Data Integration?](https://www.ibm.com/think/topics/data-integration)*

**In one line:** Data integration combines data from many sources and reconciles it into one consistent, usable form for analytics, operations and decision-making.

**Key points**
- The problem it solves: data spread across databases, apps, files, SaaS tools and APIs in different formats and quality levels, which creates silos and conflicting numbers.
- A typical flow: identify sources → extract → map fields between systems → validate → transform (cleanse, enrich, normalise) → load (batch or real time) → keep in sync → govern and secure → manage metadata → access and analyse.
- Data mapping matters because different systems use different names, codes and structures for the same thing.
- Main styles: ETL, ELT, real-time integration and CDC, application (API) integration, data virtualization and federated querying (data stays in place and queries run across sources, which saves copies but can be slow).
- IBM separates integration from ingestion: ingestion is just one step, while integration runs through to analysis and BI, so it carries more responsibility for the final result.
- Tool categories: ETL tools, ESB/middleware, replication, virtualization, iPaaS, streaming integration, data quality/governance, CDC, master data management (MDM) and API management.
- Use cases: data warehouses, data lakes, customer 360 views, BI reporting and IoT data.

**Compare / don't confuse with:** Data ingestion (moving data in) vs. data integration (moving it in *and* reconciling it into one coherent model).

**Interview angle**
- *Q: How would you choose an integration approach for a new source?* — Start from latency needs, volume, how often the source changes and how much it can be queried. Then pick batch ETL/ELT, CDC/streaming, API integration or virtualization, and plan for schema changes, data quality checks and governance from the start.

**Your experience hook:** Consolidated 2M+ KYC/KYB records from several source databases with CDC, using UUID surrogate keys to resolve ID and phone-number collisions between sources. That is the mapping and reconciliation work at the core of integration.

**Beyond IBM:** In practice the hard parts are identity (matching the same entity across sources), idempotent reloads and handling schema drift. The choice of tool matters much less than those three.

---

## ETL (extract, transform, load)
*Source: [IBM — What is ETL (Extract, Transform, Load)?](https://www.ibm.com/think/topics/etl)*

**In one line:** ETL pulls data from sources, cleans and reshapes it in a separate staging area, and only then loads it into the target.

**Key points**
- ETL dates from the 1970s and became the standard way to feed data warehouses. Its current form is *streaming ETL*: Kafka plus processors such as Kafka Streams, ksqlDB or Flink, transforming data while it is in motion.
- Extract: find the data points you need and the keys that join them across sources, then copy raw data into staging. Transform: cleanse, deduplicate, validate, convert units and currencies, mask regulated data, and shape it to the warehouse schema. Load: an initial full load, then incremental loads, with occasional full refreshes.
- Load types: *initial* (first full copy, done once), *incremental* (only new, changed or deleted rows since the last run) and *full* (replace everything).
- Variants: ETLT (light transform in staging, heavier transform in the warehouse), TETL (transform at the source first) and reverse ETL (push curated warehouse data back into SaaS and operational tools).
- Related methods: CDC (cuts the cost of the extract step), replication, virtualization, stream data integration (an umbrella term) and zero-ETL (query or replicate data without a pipeline).
- Benefits: better data quality, compliance (PII can be masked before it lands), works with legacy systems, and less cloud storage because data is filtered and aggregated first.
- Challenges: more upfront design and infrastructure (staging servers, business rules) and fragility when source schemas change. Schema registries help with the last one.

**Compare / don't confuse with:** ELT, where raw data lands first and is transformed inside the target. Streaming ETL is still ETL; only the timing changes.

**Interview angle**
- *Q: When would you still choose ETL over ELT?* — When sensitive data must be masked or dropped before it reaches the target, when the target has little compute (legacy or on-prem), or when you want to cut volume before paying to store it.
- *Q: How do you make an incremental ETL job safe to re-run?* — Track a high-water mark or log offset, write with upserts or partition overwrites keyed on business or surrogate keys, and commit the watermark only after the load succeeds.

**Your experience hook:** The MongoDB → Oracle ETL project had all three load modes: initial load, incremental (change streams), and full refresh.

**Beyond IBM:** Idempotency is what makes ETL dependable. Each run should give the same result if repeated, so retries and backfills are safe.

---

## ELT (extract, load, transform)
*Source: [IBM — What Is Extract, Load, Transform (ELT)?](https://www.ibm.com/think/topics/elt)*

**In one line:** ELT loads raw data into the target platform first and transforms it there, using the warehouse's or lakehouse's own compute.

**Key points**
- ELT grew from about 2010 for two reasons: data volumes surged, and cheap, elastic cloud storage and compute made in-warehouse transformation practical.
- Extract from almost anything (SQL/NoSQL, CRM, ERP, APIs, files, web pages). Load into a warehouse, lake or lakehouse with only light cleanup. Transform in place, usually with SQL and dbt.
- Benefits: data arrives faster; the pipeline is more resilient (a failed transform doesn't block loading); it handles semi-structured and unstructured data; raw data stays available so teams can rebuild or reuse it; it scales; and upfront cost is low.
- Challenges: raw data lands untransformed, so quality, compliance and security need governance (encryption, RBAC). Compute and storage bills grow with volume. On-prem or legacy setups often suit ETL better.
- ELT started out as batch but can also run on streams (Kafka topics loaded continuously and transformed in the target).
- dbt is described as the standard transformation tool: layered, reusable models run in dependency order.
- Use cases: AI/ML feature data, customer analytics, regulatory reporting (with audit trails back to raw data), supply chain/IoT.

**Compare / don't confuse with:** ETL (transform before load). The difference is *where and when* the transform runs, not whether it happens.

**Interview angle**
- *Q: What is the main risk of ELT and how do you control it?* — Raw, possibly sensitive and messy data sits in the warehouse. Control it with a restricted raw layer, masking or pseudonymisation, tests on each model (dbt tests), and cost monitoring on transformation queries.

**Your experience hook:** The DataTalksClub Zoomcamp's BigQuery + dbt modules follow this pattern: load raw data, then build staging and mart models in the warehouse.

**Beyond IBM:** Most teams organise ELT in layers (raw/bronze → cleaned/silver → business marts/gold). The raw layer is what makes reprocessing and backfills possible.

---

## Change data capture (CDC)
*Source: [IBM — What Is Change Data Capture?](https://www.ibm.com/think/topics/change-data-capture)*

**In one line:** CDC detects inserts, updates and deletes in a source database and passes just those changes downstream, usually with low latency.

**Key points**
- **Why:** it moves only what changed, not whole datasets, so it puts less load on the source and the network and keeps targets fresh. It enables near-zero-downtime migrations, real-time analytics, cheaper ETL (incremental extract) and up-to-date RAG/AI data.
- **Push vs. pull:** in push CDC the source side emits changes as they happen (log readers, brokers such as Kafka). In pull CDC the target polls on a schedule, which is simpler but has higher latency and adds query load on the source.
- **Three methods IBM lists:**
  - *Log-based*: reads the database transaction log (Postgres WAL via logical decoding, MySQL binlog, SQL Server log/CDC tables, Oracle redo, MongoDB oplog via change streams). Every event has an ordered log position such as an LSN. This is the most efficient and complete method, but log formats differ by database.
  - *Timestamp/query-based*: polls `WHERE updated_at > :last_seen`. Easy to set up, but it puts repeated load on the source and **cannot see hard deletes**, because the row and its timestamp are both gone.
  - *Trigger-based*: triggers write each change to a shadow/change table. It captures deletes, but adds write overhead to every transaction on the source.
- **Deletes:** log-based CDC emits an explicit delete event, often with the old row ("before image"). How much of the old row you get depends on settings: by default Postgres logs only the key unless you set `REPLICA IDENTITY FULL`, and MongoDB deletes carry only the `_id` unless pre-images are enabled. Query-based CDC needs soft-delete flags or periodic key reconciliation instead.
- **Ordering:** the log gives a total order per source (by LSN, binlog position or oplog timestamp). Downstream you usually only need *per-key* order, so partition Kafka/Redpanda topics by primary key and apply changes idempotently (upsert, and skip an event older than the version already stored).
- **Resume positions / offsets:** the connector stores the last processed position (LSN, binlog file + position or GTID, MongoDB resume token) and restarts from it. Delivery is usually at-least-once, so sinks must handle replays. If the source's log retention passes the saved position (oplog rolled over, WAL removed), you must re-snapshot. Watch the other side too: an idle Postgres replication slot makes the source keep WAL and fill its disk.
- **Snapshot handoff:** to bootstrap, take a consistent snapshot, note the log position at snapshot time, then stream from that position. Overlapping events are harmless if the sink upserts by key. Newer tools also run chunked or incremental snapshots alongside streaming.
- **CDC vs. ETL and SCD:** CDC is often the incremental feed after an initial ETL load. CDC *captures* changes; slowly changing dimensions (Type 1 overwrite, Type 2 new row with history) decide how the warehouse *stores* them.
- Tools: Debezium (open source, Kafka Connect), Oracle GoldenGate, AWS DMS, SQL Server native CDC. Selection criteria include latency, connector support, Kafka integration, reliability and deployment model.

**Compare / don't confuse with:** CDC is a *capture mechanism*; real-time integration is the wider architecture. CDC vs. replication: replication keeps a copy in sync, while CDC exposes the change events themselves for any consumer. CDC vs. CEP: CDC reports changes, and CEP finds patterns in them.

**Interview angle**
- *Q: Log-based vs. query-based vs. trigger-based CDC?* — Log-based reads the transaction log: low overhead, captures every change including deletes, in commit order, but needs log access and per-database connectors. Query-based polls a timestamp column: simple, but it misses hard deletes and intermediate updates between polls, and can miss rows committed late with an older timestamp. Trigger-based captures everything but slows every write on the source.
- *Q: How do you handle deletes end to end?* — Capture them as explicit events, carry the key (and the before image if needed), then in the sink either hard-delete, soft-delete with a flag and timestamp, or close the SCD2 row. For Kafka log compaction, a tombstone (null value) for the key lets the topic eventually drop it.
- *Q: Your CDC stream stopped but nothing alerted. What do you monitor?* — Lag between source log position and committed offset, time since last event per table, connector task state, replication slot or oplog headroom, and dead-letter queue (DLQ) counts. Alert on "no progress" as well as on errors.
- *Q: How do you start CDC on a table that already has data?* — Record the log position, take a consistent snapshot, load it, then stream from the recorded position with idempotent upserts so overlap doesn't create duplicates.

**Your experience hook:** You ran CDC to consolidate 2M+ KYC/KYB records, added resume tokens and monitoring after streams stalled silently, and built MongoDB change-stream ETL into Oracle handling inserts, updates and deletes plus initial-load and full-refresh modes. Use these as your main worked examples.

**Beyond IBM:** IBM leaves out schema changes (DDL) mid-stream and the "dual write" problem. Applications that write to a database and publish events separately should use the transactional outbox pattern, with CDC reading the outbox table.

---

## Real-time data integration
*Source: [IBM — What Is Real-Time Data Integration?](https://www.ibm.com/think/topics/real-time-data-integration)*

**In one line:** Real-time integration keeps moving new or changed data from sources to targets so it is available within milliseconds to seconds, not at the next batch run.

**Key points**
- Architecture: sources → capture (APIs, logs, CDC, connectors, event streams) → transport and buffering (Kafka, Pub/Sub) → stream processing (e.g. Flink, which keeps state across events) → targets (apps, warehouses, lakes, AI) plus governance and observability.
- Techniques: data streaming, CDC, application/API integration (not every API call is real time) and event-driven integration, where producers publish events and consumers react independently.
- It is not a rival to ETL/ELT: a CDC → Kafka → Flink → warehouse pipeline is both real-time *and* ETL-shaped.
- Good fit when data loses value quickly: payments and fraud, industrial monitoring, inventory, personalisation, AI agents that need the current account state. Batch still suits reconciliation, periodic reporting and large scheduled transforms; many firms run both.
- Challenges: running many systems continuously, less time for quality checks, duplicates and out-of-order events, schema evolution, always-on cost, and data copies spreading (GDPR minimisation, CCPA).
- Best practices: set the latency target first, separate ingest, process and serve stages, decide on a duplicate/late-data policy, design for replay, monitor end-to-end latency, validate schemas, and check quality early.

**Compare / don't confuse with:** CDC (one way to source changes) and data streaming (the movement of data). Real-time integration is the end-to-end architecture that uses both.

**Interview angle**
- *Q: A stakeholder asks for "real time". What do you ask first?* — What decision depends on it and how stale the data can be before that decision suffers. Seconds, minutes and hours lead to very different costs and designs, and micro-batch is often enough.

**Your experience hook:** The current Airflow + Redpanda pipelines and the MongoDB change-stream → Oracle sync are both real-time integration with a batch side for initial loads and refreshes.

**Beyond IBM:** Exactly-once is usually achieved end to end as "at-least-once delivery + idempotent or transactional sink", not as a property of the transport alone.

---

## Cloud data integration
*Source: [IBM — What Is Cloud Data Integration?](https://www.ibm.com/think/topics/cloud-data-integration)*

**In one line:** Cloud data integration is data integration where at least one source or target runs in the cloud, including hybrid (cloud + on-prem) and multicloud setups.

**Key points**
- Two subtypes: *hybrid* (public cloud, private cloud and on-prem) and *multicloud* (several cloud providers). Most large firms are both.
- Driver: data is now split across cloud and on-prem (on-prem often kept for regulation or performance), which fragments the data landscape.
- Steps are the usual ones, with cloud traits: sources include SaaS and object stores, extraction goes through APIs and managed connectors, mapping must cope with schema drift in SaaS sources, and **ELT is the dominant loading pattern**.
- Benefits: unified access (backed by catalogs and metadata), quality and consistency, scalability via storage/compute separation and elastic compute, faster innovation and AI enablement.
- Challenges: more attack surface and data residency rules (GDPR, HIPAA, PCI DSS); performance and cost at volume (parallelism, partitioning, monitoring); real-time over distributed networks; hybrid deployments where regulated data must stay on-prem.
- "Remote engine" execution: design pipelines centrally but run them where the data lives, which cuts egress cost and latency.

**Interview angle**
- *Q: What changes when an on-prem pipeline moves to the cloud?* — Network and egress costs, identity and secrets management, data residency, elastic compute (favours ELT), and managed services that bring their own quotas and limits. Processing near the data often matters more than raw speed.

**Beyond IBM:** Egress fees and cross-region transfer are often the biggest hidden cost. Co-locate compute with storage and compress or filter before moving data.

---

## AI data integration
*Source: [IBM — What Is AI Data Integration?](https://www.ibm.com/think/topics/ai-data-integration)*

**In one line:** AI data integration uses machine learning, LLMs and agents to automate parts of building and running data pipelines, such as discovery, mapping, transformation and monitoring.

**Key points**
- Why now: unstructured data growth (often cited as ~90% of enterprise data), LLMs and agents need unified trusted data, real-time decisions need faster pipelines, and hand-built pipelines break as complexity grows.
- Problems targeted: slow data access (requests can take 1–4 weeks), tool sprawl, silent failures from schema changes, engineers spending much of their time on maintenance, and a shortage of skilled staff.
- Uses: discovering and classifying data, enriching metadata, extracting structure from documents, **semantic schema mapping** (e.g. matching `emp_ID` to `employee_number`), generating transformation logic, anomaly and schema-drift detection, root-cause hints, and natural-language-to-SQL for business users.
- Agentic pipeline design: you declare outcomes and governance rules, the system proposes a plan (ETL/ELT, streaming, replication) for human approval, then agents execute it.
- Things to evaluate in a platform: interoperability and extensibility (connectors, custom code), security and governance, and deployment flexibility (hybrid, in-place processing).

**Interview angle**
- *Q: Where would you trust an LLM in a data pipeline, and where not?* — It works well for drafting mappings, docs, tests and SQL that a human reviews, and for flagging anomalies. Don't let it make unreviewed changes to production transformations or access rules; keep deterministic tests and approval gates.

**Your experience hook:** You've built n8n workflows that orchestrate Claude Code agents, and customer-facing RAG systems, so you can speak to both sides: agents doing integration work, and integration feeding AI.

**Beyond IBM:** Generated SQL and mappings still need the same version control, tests and review as hand-written code. Without them, errors spread faster.

---

## Data interoperability
*Source: [IBM — What Is Data Interoperability?](https://www.ibm.com/think/topics/data-interoperability)*

**In one line:** Data interoperability is the ability of different systems to exchange data and correctly use it, thanks to shared formats, meanings and protocols.

**Key points**
- Interoperable data has a consistent format, can be combined with other data, is shared through APIs, and often follows industry standards (EDI, FHIR).
- Levels: *foundational/technical* (data can be transported), *structural/syntactic* (formats both sides can parse), *semantic* (both sides agree what the data means) and *organisational* (aligned policies and governance).
- Methods: standardise formats (JSON, XML), share vocabularies (e.g. LOINC lab codes), manage metadata, use APIs for transport, adopt frameworks (DICOM, EDI, FHIR, ISO/IEC 21823-4 for IoT, P25 for radio), and support it with governance.
- Benefits: more data accessible and used (much enterprise data sits unused), agility, efficiency, collaboration, easier scaling when adding systems, and compliance (internal APIs helped with GDPR tracking).
- Challenges: legacy formats, poor data quality spreading to other systems, and change management/training.
- Use cases: financial services, government, healthcare (EHR interoperability is mandated in the US for Medicare/Medicaid), supply chains.

**Compare / don't confuse with:** Integration (the process of combining data) vs. interoperability (the property that lets systems exchange and understand data without custom glue each time).

**Interview angle**
- *Q: Two systems both send JSON but the data still doesn't line up. Why?* — That is syntactic interoperability without semantic interoperability: different meanings, units, codes or identifiers for the same field. Fix it with shared definitions, reference data and a canonical model.

**Your experience hook:** Merging KYC data across source databases where IDs and phone numbers collided was a semantic interoperability problem, and you solved it with UUID surrogate keys.

**Beyond IBM:** In payments and banking the relevant standard is ISO 20022 (structured financial messaging). Within a company, a canonical data model plus a schema registry does the same job.

---

## Data contract
*Source: [IBM — What Is a Data Contract?](https://www.ibm.com/think/topics/data-contract)*

**In one line:** A data contract is a code-defined agreement between data producers and consumers about a dataset's schema, quality, meaning and availability, which tooling can enforce automatically.

**Key points**
- The problem: producers (often application engineers) change schemas for their own needs, and consumers' pipelines, dashboards and models break. Something as small as dropping one column can cause this.
- Contents: fundamentals (ID, version, status, purpose, legal limits), schema, quality rules (completeness, validity, null counts), support channels, pricing, owning team, access roles, SLAs (freshness, retention) and infrastructure/servers. Custom properties can be added.
- Format: YAML or JSON, so it is machine-readable. Many follow the **Open Data Contract Standard (ODCS)** from the Bitol project (Linux Foundation).
- Enforcement flow: consumers state their needs → producers agree → contract written in code → schema changes go through pull requests → CI checks the change against the contract → violations alert the producer before release.
- Benefits: data quality, managed dependencies, versioning (time to adapt to changes), collaboration, accountability, governance, scalability.
- Data contracts underpin *data products* and *data mesh*: they act as the quality guarantee each domain's data product publishes.
- Tooling: Avro/Protobuf for schemas, Great Expectations/dbt tests for quality, schema registries for compatibility checks, governance platforms for lineage and catalog.

**Compare / don't confuse with:** A schema alone (structure only) vs. a contract (structure + quality + SLA + ownership). The usual analogy: a data contract is to data what an API spec is to services.

**Interview angle**
- *Q: How would you introduce data contracts without stalling producers?* — Start with the few critical datasets, put the contract in the producer's repo, run compatibility checks in their CI, and version breaking changes with a deprecation window rather than blocking every change.

**Beyond IBM:** Most of the practical value comes from automated schema-compatibility checks (backward/forward) in CI and the registry. The written agreement matters less than that gate.

---

## Data integration techniques and methods
*Source: [IBM — Data Integration Techniques and Methods](https://www.ibm.com/think/topics/data-integration-techniques-methods)*

**In one line:** An overview of the main ways to combine data, each with a key advantage and a key challenge.

**Key points**
- Steps across all techniques: identify, map, transform, validate, load, synchronise.
- **ETL:** clean before loading (best for quality), but traditional versions struggle with volume and streams.
- **ELT:** fast ingestion of every data type, but needs strong transformation and governance to avoid a messy target.
- **Real-time integration:** fresh data for AI and decisions, but needs infrastructure able to handle the velocity.
- **CDC:** efficient and up to date with low resource use, but **schema changes can break pipelines**.
- **Data virtualization / federation:** query data in place through a virtual layer with no copies, but adds query latency, especially with frequent updates.
- **Application integration:** connects apps through APIs, middleware and webhooks, but legacy-to-SaaS links are hard.
- **Data replication:** keeps copies for availability, disaster recovery and locality, but it's hard to balance freshness against system load. It can be synchronous or asynchronous.
- Agentic integration: natural-language requests ("combine CRM and ERP data") handled by coordinated agents.

**Compare / don't confuse with:** Virtualization (no data movement; compute at query time) vs. replication (copies data; compute at write time).

**Interview angle**
- *Q: Synchronous vs. asynchronous replication?* — Synchronous waits for replicas to confirm before the write commits: no data loss on failover, but higher write latency. Asynchronous commits locally and ships changes afterwards: faster, but replicas lag and recent writes can be lost on failover.

**Beyond IBM:** IBM describes async replication as "batches with a delay". In most databases it is a continuous log stream that simply doesn't wait for replica acknowledgement.

---

## Self-test

1. What is the difference between data ingestion and data integration?
<details><summary>Answer</summary>Ingestion collects data and makes it available in a destination. Integration includes ingestion but also maps, cleans, reconciles and harmonises data from several sources into one consistent model fit for analysis.</details>

2. Give two situations where ETL is a better choice than ELT.
<details><summary>Answer</summary>When regulated or sensitive data must be masked or removed before it reaches the target, and when the target is a legacy or on-prem system without much compute. Cutting volume before paying for cloud storage is a third.</details>

3. Define initial, incremental and full loads.
<details><summary>Answer</summary>Initial: the first complete copy of a source. Incremental: only rows that are new, changed or deleted since the last run. Full: transfer the whole dataset again, usually replacing what's in the target.</details>

4. Why can't timestamp/query-based CDC see hard deletes, and what are the workarounds?
<details><summary>Answer</summary>The query looks for rows with a newer `updated_at`, but a deleted row and its timestamp no longer exist. Workarounds: soft deletes (a deleted flag and timestamp), triggers writing to a change table, periodic key reconciliation between source and target, or switching to log-based CDC.</details>

5. How does a CDC pipeline resume after a crash without losing or duplicating data?
<details><summary>Answer</summary>It stores the last processed log position (LSN, binlog position/GTID, MongoDB resume token) and restarts from it. Delivery is at-least-once, so the sink must be idempotent (upsert by key, ignore older versions). If log retention has removed the saved position, the table must be re-snapshotted.</details>

6. Describe the snapshot-to-stream handoff when starting CDC on an existing table.
<details><summary>Answer</summary>Record the current log position, take a consistent snapshot and load it, then stream changes from the recorded position. Events that overlap with the snapshot are harmless because the sink applies them as idempotent upserts.</details>

7. How do you keep CDC events in the right order downstream?
<details><summary>Answer</summary>The source log is ordered (LSN/oplog). Partition the topic by primary key so each key's events stay in order within one partition, and make the sink compare versions or positions so a late, older event can't overwrite a newer one.</details>

8. How do CDC and slowly changing dimensions relate?
<details><summary>Answer</summary>CDC captures and delivers changes; the SCD type decides how the warehouse stores them. Type 1 overwrites and keeps no history. Type 2 adds a new row with validity dates, keeping full history.</details>

9. What are the four interoperability levels IBM lists?
<details><summary>Answer</summary>Foundational/technical (data can be transported), structural/syntactic (common formats), semantic (shared meaning and vocabularies) and organisational (aligned policies and governance).</details>

10. What does a data contract contain, and how is it enforced?
<details><summary>Answer</summary>Schema, quality rules, SLAs, ownership/team, access roles, support channels, version/status and server details, written in YAML/JSON (often ODCS). It is enforced by checks in CI/CD on producer changes and by schema registries, so breaking changes are caught before they reach consumers.</details>
