# 9. Data processing

> This section is about what happens to data once you have it: turning it into information (processing), getting it back out (retrieval), bringing it in (ingestion), analysing it across dimensions (OLAP) and handling it continuously as streams (stream processing, real-time streaming, complex event processing, streaming analytics), plus processing unstructured content. Interviewers use these topics to probe batch vs. streaming trade-offs, windowing and late data, OLTP vs. OLAP, indexing and query plans, and how you would feed a RAG system.

## Data processing
*Source: [IBM — What Is Data Processing?](https://www.ibm.com/think/topics/data-processing)*

**In one line:** Data processing turns raw data into usable information through a series of steps from collection to storage.

**Key points**
- Six stages: collection → preparation (clean, validate, standardise) → input (into an engine such as Spark) → analysis → output (dashboards, reports) → storage (warehouse, lake, cloud).
- It has moved from manual work, to electronic data processing, to parallel and cloud processing with ML/AI.
- Why it matters: better forecasting and decisions, BI, compliance (GDPR-aligned storage and access), and clean data for gen AI. Poor data makes AI outputs unreliable.
- Technologies: cloud and big data frameworks (Hadoop, Spark), ML automation, Kafka for real-time pipelines, edge computing (process near the source for low latency), and quantum computing (still emerging).
- Types IBM lists: batch, real-time, multiprocessing (parallel across CPUs), manual (where human judgement is needed) and online (continuous interactive updates).
- Challenges: data quality (IBM cites Unity Software losing roughly USD 5bn in market value over bad data), scalability, integration complexity (data prep is still the biggest time sink) and regulation (GDPR fines of about EUR 1.2bn in 2024).

**Compare / don't confuse with:** Batch (scheduled, high throughput, cheaper) vs. real-time/stream (continuous, low latency, costlier to run).

**Interview angle**
- *Q: What makes a batch processing job production-grade?* — It is deterministic and idempotent (safe to re-run), partitioned by time or key so you can backfill, validated on input and output, monitored for freshness and row counts, and its outputs are written atomically.

**Your experience hook:** Credit scoring on MongoDB aggregation pipelines with factor analysis and PCA is the full prepare → analyse → output chain applied to a real decision.

**Beyond IBM:** The useful axis is latency: batch, micro-batch or streaming. "Multiprocessing" and "manual" are ways of doing the work, not processing models in the same sense.

---

## Data retrieval
*Source: [IBM — What Is Data Retrieval?](https://www.ibm.com/think/topics/data-retrieval)*

**In one line:** Data retrieval means getting the information you need out of a data store, from exact SQL lookups to semantic and vector search over unstructured content.

**Key points**
- Traditional techniques: query languages (SQL), **indexing** (smaller searchable structures that point to rows, avoiding full scans) and **query optimisation** (the optimiser chooses a plan: whether to use an index, access path, join order).
- Where they fall short: unstructured data, capturing meaning, scale and real-time results.
- AI techniques: **vector search** (embeddings plus nearest-neighbour lookup), ML/deep learning (query suggestions, image search with CNNs), NLP semantic search (matches intent, not just keywords), and **RAG / agentic RAG**.
- Agentic RAG components: hybrid keyword + vector search, semantic caching, agentic chunking (semantically coherent chunks), routing agents (choose the source or tool) and query-planning agents (split a question into sub-queries and merge the answers).
- Data retrieval returns exact matches; information retrieval (IR) returns ranked, relevance-scored results (web search). Data mining is different again: it finds patterns, not just records.
- Challenges: retrieved data may be low quality, security (stopping the wrong people from retrieving sensitive data) and vendor lock-in.

**Compare / don't confuse with:** Data retrieval (exact) vs. information retrieval (ranked relevance) vs. data mining (pattern discovery).

**Interview angle**
- *Q: A query got slow as the table grew. How do you investigate?* — Run `EXPLAIN ANALYZE` and look for sequential scans, bad row estimates and expensive joins or sorts. Add or adjust indexes (column order and selectivity matter), refresh statistics, then consider partitioning or rewriting the query.
- *Q: Why use hybrid search in RAG?* — Keyword search (e.g. BM25) catches exact terms, IDs and rare words; vector search catches paraphrases. Combining and re-ranking them beats either alone.

**Your experience hook:** You tuned PostgreSQL to about 900 req/s (indexes and query plans), and you've built customer-facing RAG systems (vector and hybrid retrieval), so you cover both halves of this topic.

**Beyond IBM:** Vector indexes such as HNSW are *approximate*: they trade a little recall for speed. Access control must be applied at retrieval time, so RAG never returns chunks the user isn't allowed to see.

---

## Data ingestion
*Source: [IBM — What is Data Ingestion?](https://www.ibm.com/think/topics/data-ingestion)*

**In one line:** Data ingestion collects data from source systems and lands it somewhere it can be stored, processed and analysed.

**Key points**
- Ingestion is the first step of any pipeline. Sources include financial systems, SaaS, ERP/CRM, IoT, social media and third-party feeds, in structured, semi-structured and unstructured form.
- Steps: discovery → acquisition → validation (type, range, uniqueness checks) → transformation (normalise, aggregate, standardise) → loading (batch or real time). Simple pipelines may skip transformation.
- Types: **batch**, **real-time**, **CDC**, **streaming ingestion** (capture continuously; stream *processing* analyses continuously), **API-based** (rate limits, auth and availability affect it), **micro-batching** (small frequent batches) and **Lambda architecture** (batch layer + speed layer + serving layer).
- Tool choices: open source (Kafka), proprietary, cloud-managed (AWS Glue, Kinesis, Snowflake) or on-prem. Build approaches: hand-coded, prebuilt connectors, or full integration platforms.
- Challenges: security, scale and variety, ingestion sprawl and cost, fragmentation, **schema drift** and quality assurance.
- Use cases: cloud lakes and lakehouses, cloud migration (with CDC to keep systems in sync), real-time analytics, feeding observability, AI/RAG, customer 360, BI.

**Compare / don't confuse with:** Ingestion (getting data in) vs. integration (reconciling it into one model). ETL and ELT are specific ways of ingesting that differ in *when* transformation happens.

**Interview angle**
- *Q: How do you design ingestion that copes with retries and re-deliveries?* — Land raw data unchanged with load metadata (source, batch ID, offset, ingest time), make loads idempotent using natural or surrogate keys, quarantine bad records in a DLQ instead of failing the whole run, and alert on freshness and volume anomalies.

**Your experience hook:** Airflow + Redpanda ingestion into analysis-ready and ML datasets, and RabbitMQ/Redis queues with idempotency, retries and DLQs, map directly onto these patterns.

**Beyond IBM:** Lambda architecture means maintaining the same logic twice (batch and speed). Many teams now prefer Kappa (one streaming path with replay from the log), or a lakehouse with incremental processing.

---

## OLAP (online analytical processing)
*Source: [IBM — What is OLAP?](https://www.ibm.com/think/topics/olap)*

**In one line:** OLAP runs fast, complex, multidimensional analytical queries (for example sales by region × product × time) over large historical datasets.

**Key points**
- Business data has *dimensions* (time, location, product), each with hierarchies (year → quarter → month). A relational table shows two dimensions at a time; OLAP rearranges data so you can analyse many at once.
- **OLAP cube:** a multidimensional array of pre-aggregated measures. With more than three dimensions it is sometimes called a hypercube.
- Operations: **drill-down** (more detail, or add a dimension), **roll-up** (aggregate upward), **slice** (fix one dimension to a single value), **dice** (choose a sub-cube across several dimensions), **pivot** (rotate the view).
- Types: **MOLAP** (true cubes; fastest), **ROLAP** (works directly on relational tables; handles large data, but SQL gets complex), **HOLAP** (cubes for aggregates, relational tables for detail; drill-through, but more complex and costly).
- **OLAP vs. OLTP:** OLAP is analytical, historical and read-heavy (BI, forecasting, budgeting). OLTP handles fast, accurate small transactions (ATMs, payments, bookings).
- Cloud MPP warehouses make OLAP cheaper and scalable, so analysis runs where the data already sits.

**Compare / don't confuse with:** OLTP (row-oriented, normalised, many small reads and writes, ACID) vs. OLAP (columnar, often star schemas, large scans and aggregates).

**Interview angle**
- *Q: Why shouldn't you run heavy analytics on the production OLTP database?* — Long scans and aggregates compete with short transactions for CPU, I/O and locks, and the row-oriented, normalised layout is inefficient for them. Replicate or CDC the data into an analytical store instead.

**Your experience hook:** The Zoomcamp BigQuery and dbt work is modern ROLAP: columnar MPP queries over star-schema-like models with no cube build step.

**Beyond IBM:** Standalone MOLAP cubes are now niche. Columnar warehouses, `GROUP BY ROLLUP/CUBE/GROUPING SETS` and semantic layers do most of the job, with materialised aggregates where speed matters.

---

## Stream processing
*Source: [IBM — What Is Stream Processing?](https://www.ibm.com/think/topics/stream-processing)*

**In one line:** Stream processing analyses and transforms unbounded flows of events continuously as they arrive, rather than in scheduled batches.

**Key points**
- Each change is an event, so it is also called event stream processing. It is used for fraud, personalisation, monitoring, and fresh features for AI/ML (feature stores, model monitoring).
- Architecture patterns: **Lambda** (separate batch and streaming pipelines) and **Kappa** (a single streaming pipeline for everything).
- **Streaming platforms** (Kafka, Confluent, Kinesis, Pub/Sub, Event Hubs) transport and store events in a durable, replayable log. **Processing frameworks** (Kafka Streams, ksqlDB, Flink, Spark Structured Streaming, Storm) do the computing: filter, transform, join, aggregate.
- Three stages: ingest (unbounded input via connectors such as Kafka Connect) → process (aggregate, enrich, join streams, ML inference) → output (dashboards, databases, alerts, lake/warehouse).
- Challenges: volume and velocity (scale out), deciding what must be real-time vs. stored, reliable APIs and logic, language choice (Java for production in Kafka/Flink; Python for prototyping/ML), **schema governance**, and combining live and historical queries.
- Batch remains simpler and cheaper for reporting and historical analysis; hybrid setups are common.

**Compare / don't confuse with:** Streaming platform (Kafka: the durable log and transport) vs. stream processor (Flink: stateful computation).

**Interview angle**
- *Q: Event time vs. processing time, and why does it matter?* — Event time is when the event happened; processing time is when the system sees it. Correct windowed results need event time plus *watermarks* that set how long to wait for late events, and a policy for events that arrive later still.
- *Q: How does Flink provide exactly-once?* — It takes periodic checkpoints of operator state and source offsets, and uses transactional or idempotent sinks, so after a failure it restores state and replays from the checkpointed offsets.

**Your experience hook:** You run Redpanda (Kafka-compatible) in production ETL and covered Kafka/Flink in the Zoomcamp.

**Beyond IBM:** IBM doesn't mention state size, checkpointing, watermarks or back-pressure, which are the concepts interviewers usually dig into.

---

## Unstructured data processing
*Source: [IBM — What Is Unstructured Data Processing?](https://www.ibm.com/think/topics/unstructured-data-processing)*

**In one line:** Unstructured data processing extracts structure and meaning from content with no predefined model (text, documents, images, audio) so analytics and AI can use it.

**Key points**
- Unstructured data is commonly cited as ~90% of enterprise data. It holds context (tone, imagery) that tables can't, and it is the raw material for LLMs.
- Three categories: structured (fixed schema, SQL), semi-structured (tags or metadata such as JSON/XML; IBM also lists CSV) and unstructured (no model).
- Pipeline stages, adapted: collection (into lakes, object storage, NoSQL; streaming or API) → preparation (OCR, entity recognition, NLP keywords and sentiment, semantic tagging, enrichment) → input (connectors and stream tools, keeping lineage and access controls) → analysis (ML, NLP, data mining instead of SQL) → output (dashboards mixing structured and unstructured results) → storage (S3, Blob, governed lakes).
- There is a feedback loop: AI enriches unstructured data, and the enriched data trains better models.
- Challenges: no schema (use metadata and semantic tagging), quality and duplicates, volume and cost (tiering), mixing RDBMS and NoSQL, latency, and skills/literacy.

**Compare / don't confuse with:** Structured processing (schema-on-write, SQL) vs. unstructured processing (schema-on-read, extraction by ML/NLP).

**Interview angle**
- *Q: Walk me through preparing a document corpus for RAG.* — Parse/OCR → clean and remove boilerplate → detect and redact PII → chunk by structure or meaning with overlap → attach metadata (source, date, access labels) → embed → index (vector + keyword) → evaluate retrieval quality, and re-embed when the model changes.

**Your experience hook:** Building customer-facing RAG systems is exactly this pipeline, and PII pseudonymisation from the KYC work shows where redaction belongs in it.

**Beyond IBM:** Keep the raw documents and record which parser, chunker and embedding model versions were used, so you can rebuild the index reproducibly.

---

## Real-time data streaming
*Source: [IBM — What Is Real-Time Data Streaming?](https://www.ibm.com/think/topics/real-time-data-streaming)*

**In one line:** Real-time data streaming processes each data point as it arrives, often milliseconds after it is created, instead of waiting for a batch.

**Key points**
- Sources: IoT sensors, market feeds, social media, POS/e-commerce, clickstreams. Uses: dynamic pricing, fraud detection, predictive maintenance, and agentic AI that acts on live data.
- Benefits: fresher insights, efficiency, avoiding data hoarding (filter early so less redundant data is stored), predictive analytics that combine live and historical data, and risk management.
- IBM distinguishes **event streaming** (which filters before moving data, making time-series and signal processing harder) from real-time data streaming, but notes they use the same platforms: Kafka, Kinesis, **Redpanda**.
- Architecture: ingestion (unbounded data via streaming connectors/APIs) → processing (filter, enrich, transform, ML) → destination (an app, or storage in a lake or lakehouse, or a warehouse).
- Technology: Kafka (pub/sub log), Flink (stateful processing and CEP), Spark Streaming (fault-tolerant micro-batch), managed services (Kinesis, Confluent, Azure Stream Analytics, Dataflow).
- Considerations: cost against the cost of stale data, **fault tolerance** (avoid data loss), observability (schema changes, drift) and security of PII in motion.

**Compare / don't confuse with:** Batch processing (scheduled intervals, efficient for routine reports) vs. streaming (continuous, per event).

**Interview angle**
- *Q: What delivery guarantees exist in streaming and which would you pick for payments data?* — At-most-once (may lose), at-least-once (may duplicate) and effectively exactly-once (at-least-once plus idempotent or transactional processing). For money, use at-least-once with idempotency keys and deduplication so nothing is lost and nothing is applied twice.

**Your experience hook:** Redpanda, which IBM names as a leading streaming platform, is the backbone of your current ETL.

**Beyond IBM:** Most practitioners treat "event streaming" and "data streaming" as the same thing. Don't lean on IBM's distinction in an interview.

---

## Complex event processing (CEP)
*Source: [IBM — What is Complex Event Processing?](https://www.ibm.com/think/topics/complex-event-processing)*

**In one line:** CEP detects meaningful patterns across many events over time, possibly from several streams, and triggers an action when a rule matches.

**Key points**
- IBM names three event-processing patterns: *simple* (one event → one action), *event stream processing* (filter or aggregate one ordered stream) and *complex* (combinations across streams within a time window).
- Components: ingestion (Kafka as the ordered, durable log) → **stateful window management** (Flink; exactly-once state) → pattern/rule evaluation (ksqlDB makes it SQL-accessible) → aggregation and correlation across streams → automated response (alert, API call, new event).
- Window types: **tumbling** (fixed and non-overlapping; can miss patterns that span a boundary), **sliding** (moves continuously; holds the most state) and **session** (closes after a period of inactivity). Sizing windows is hard: too narrow misses slow patterns, too wide costs memory and raises false positives.
- CEP can fire on **non-occurrence**, such as a payment initiated but never confirmed, or a login not followed by MFA.
- How ML fits: ML finds new patterns in historical data, and CEP encodes them as rules and applies them live.
- Related but different: ESP (single stream), CDC (produces the change events that CEP can consume), BRMS (rules on current state vs. CEP's rules over time), anomaly detection (is this *value* odd? vs. is this *sequence* suspicious?).
- Challenges: maintaining rules (ownership, versioning, testing, retirement), latency vs. throughput vs. state size, scaling, and the cost of stateful processing.

**Compare / don't confuse with:** CDC vs. CEP: CDC faithfully reports what changed; CEP interprets the order, timing and causes of events.

**Interview angle**
- *Q: How would you detect "password changed, new device, then a large transfer within 5 minutes"?* — Key all three streams by account, use a CEP pattern (e.g. Flink CEP `MATCH_RECOGNIZE` or a stateful keyed function) with a 5-minute event-time bound and watermarks, and emit an alert event on match. Use timers to expire partial matches.

**Your experience hook:** CDC feeds from your lending and KYC systems are exactly the kind of input stream a CEP layer would consume. Flink windowing comes from the Zoomcamp.

**Beyond IBM:** In Flink terms, IBM's "sliding" window is a hopping window (fixed size, overlapping slide). A per-event "last N minutes" check is usually built with keyed state and timers.

---

## Streaming analytics
*Source: [IBM — What Is Streaming Analytics?](https://www.ibm.com/think/topics/streaming-analytics)*

**In one line:** Streaming analytics continuously ingests, processes and analyses data in motion so insights are available almost as soon as the data is created.

**Key points**
- It is the analytics layer of an event-driven architecture. IBM cites IDC: 63% of use cases need data processed within minutes to be useful.
- Four parts: **ingestion** (a data streaming platform with connectors and CDC; Kafka), **processing and analysis** (filter, enrich, join, aggregate; may include CEP), **governance** (lineage and masking applied in motion) and **consumption** (live dashboards, and loading to the warehouse for history and ML).
- Streams are often written to open table formats (**Apache Iceberg**, **Delta Lake**, **Apache Hudi**) so batch engines can query them.
- AI works in both directions: real-time inference runs inside the stream, and streaming insights feed agents and chatbots with current context.
- Streaming analytics describes the *process*; real-time analytics describes the *result* (low-latency insight), and some definitions include micro-batching.
- Benefits: faster decisions, scale, CX, AI, efficiency, security monitoring. Challenges: integration, quality, governance, scale, speed.
- Governance tools: Apache Atlas, DataHub, Confluent Stream Governance (schemas, tags, lineage).

**Compare / don't confuse with:** Streaming analytics vs. real-time analytics (process vs. outcome); streaming analytics vs. batch BI (insight in seconds vs. hours later).

**Interview angle**
- *Q: How do you serve both a live dashboard and historical analysis from the same stream?* — Compute live aggregates in the stream processor, or in a real-time OLAP store, for the dashboard. In parallel, sink the raw and enriched events into an Iceberg/Delta table for historical and ML queries. Both read the same topic, so they stay consistent.

**Beyond IBM:** For low-latency queries over streams, teams often add a real-time OLAP store (ClickHouse, Apache Pinot, Apache Druid) between the stream and the dashboard.

---

## Self-test

1. List IBM's six stages of data processing.
<details><summary>Answer</summary>Collection, preparation, input, analysis, output and storage.</details>

2. What is the difference between data retrieval, information retrieval and data mining?
<details><summary>Answer</summary>Data retrieval returns exact matches to a query. Information retrieval returns results ranked by relevance (like web search). Data mining analyses large datasets to discover patterns rather than fetching specific records.</details>

3. What do indexes and the query optimiser each contribute to retrieval?
<details><summary>Answer</summary>Indexes are smaller searchable structures that point to rows, so queries avoid full table scans. The optimiser chooses among execution plans: whether to use an index, how to access each table and in what order to join.</details>

4. Explain Lambda vs. Kappa architecture.
<details><summary>Answer</summary>Lambda runs a batch layer (complete history) and a speed layer (recent data), merged in a serving layer, so the logic is maintained twice. Kappa uses one streaming pipeline and reprocesses history by replaying the log.</details>

5. Define slice, dice, drill-down, roll-up and pivot.
<details><summary>Answer</summary>Slice: fix one dimension to a single value. Dice: choose a sub-cube across several dimensions. Drill-down: go to more detail (down a hierarchy or add a dimension). Roll-up: aggregate upward. Pivot: rotate the view to show different dimensions on the axes.</details>

6. MOLAP, ROLAP, HOLAP: one line each.
<details><summary>Answer</summary>MOLAP uses pre-built multidimensional cubes and is fastest. ROLAP queries relational tables directly; it scales to more data but needs complex SQL. HOLAP keeps aggregates in cubes and detail in relational tables and can drill through between them.</details>

7. What is the role of Kafka vs. Flink in a streaming system?
<details><summary>Answer</summary>Kafka (or Redpanda) is the durable, partitioned, replayable event log and transport. Flink is the stateful processing engine that filters, joins, windows and aggregates events, with checkpointed exactly-once state.</details>

8. Tumbling vs. sliding vs. session windows?
<details><summary>Answer</summary>Tumbling: fixed, non-overlapping intervals. Sliding (hopping): fixed-size windows that overlap as they advance. Session: grouped by activity and closed after a gap of inactivity.</details>

9. What can CEP detect that plain stream analytics or anomaly detection can't easily detect?
<details><summary>Answer</summary>Ordered multi-event patterns across streams within a time bound, including the *absence* of an expected event (e.g. a payment initiated but not confirmed within N minutes).</details>

10. Why write streaming data into Iceberg or Delta tables?
<details><summary>Answer</summary>So batch and SQL engines can query the stream's history with ACID guarantees, schema evolution and time travel, combining real-time data with historical context for analytics and ML.</details>
