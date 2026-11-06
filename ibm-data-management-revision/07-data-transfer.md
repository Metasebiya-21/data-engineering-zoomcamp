# 7. Data transfer

> This section is about getting data from one place to another: moving files, migrating whole systems, choosing between batch and continuous movement, running streaming platforms and measuring how late data arrives. Interviewers use these topics to test whether you understand the trade-offs behind the tools: batch vs streaming, ETL vs CDC, queue vs log, at-least-once vs exactly-once, and how to prove nothing was lost or duplicated on the way.

## File transfer
*Source: [IBM — What Is a File Transfer?](https://www.ibm.com/think/topics/file-transfer)*

**In one line:** File transfer is sending files from one system to another over a network, using a protocol that defines how data is packaged, sent and checked.

**Key points**
- History: physical tapes → FTP on LANs (1970s) → email and web sharing (1990s) → encrypted SFTP/HTTPS (2000s) → managed file transfer and cloud sync.
- Flow: send request → authenticate both ends → prepare (compress, encrypt, split into packets) → transmit → error-check and resend lost parts → reassemble → verify with sizes, **checksums** or **cryptographic hashes**.
- Protocols: **FTP** (old, client/server, plain text), **SFTP** (encrypted over SSH, slower, no built-in management), **HTTP/HTTPS** (TLS-encrypted, web and cloud storage), **FASP** (UDP-based, built for fast transfer of large files over long, high-latency links) and **MFT** (a managed platform with encryption, audit trails, automation and partner management for GDPR, HIPAA, PCI DSS).
- Speed depends on bandwidth, congestion, distance, file size, protocol overhead, firewalls and ISP throttling.
- Enterprise drivers: data growth, global/remote work, regulation. Challenges: legacy integration, scaling and pricing, security threats.
- Good solutions offer visibility, security, reliability, scalability, automation (schedules and event triggers) and both UI and API/CLI access.

**Compare / don't confuse with:** SFTP (SSH File Transfer Protocol, a separate protocol running over SSH) vs. FTPS (classic FTP wrapped in TLS). IBM calls SFTP an "enhancement of FTP", but they are different protocols.

**Interview angle**
- *Q: A partner drops daily CSVs on an SFTP server. How do you ingest them safely?* — Wait for a completion signal (a marker file or atomic rename) so you never read half-written files; verify a checksum; copy the raw file to object storage; record the filename and hash in a load manifest so reprocessing the same file is a no-op; validate schema and row counts; move failed files to a quarantine folder and alert.

**Beyond IBM:** IBM lists TCP as a file transfer protocol; TCP is the transport layer underneath FTP, SFTP and HTTP. FASP is IBM Aspera's proprietary protocol. In data engineering, file drops are usually followed by object storage (S3, ADLS, GCS) as the landing zone, and idempotency comes from tracking which files were already loaded.

---

## Data migration
*Source: [IBM — What is Data Migration?](https://www.ibm.com/think/topics/data-migration)*

**In one line:** Data migration is moving data from one storage system or computing environment to another, for example to the cloud or to a new database.

**Key points**
- Triggers: replacing servers or storage, consolidating or closing data centres, moving to public, private, hybrid or multicloud.
- Two transport options: **online** (over the internet or a dedicated WAN link) and **offline** (ship a storage appliance). IBM's rule of thumb: under ~10 TB, a client device is often simplest; multi-petabyte moves suit a provider-supplied appliance; online works in theory for any size but time limits it.
- **Database migration:** convert the schema/DDL if source and target engines differ, move the data, then set up ongoing incremental updates; you can consolidate databases at the same time.
- **Data centre migration:** large-scale; petabyte transfers can take weeks, so plan timelines early.
- Strategy factors: workload type (VMs, backups, databases have vendor tools; mission-critical data can move in stages with source and target running in parallel), data volume, speed to completion.
- Best practices: understand who uses the data and how, assess source and target (OS, schemas, dirty data), confirm business requirements (deadline, tolerable loss, security).
- Risks: security in transit and in shipping, unpredictable transfer times, cost overruns. Tools include Rclone and Cyberduck (open source) and vendor services.

**Compare / don't confuse with:** Migration (one-time move with a cutover) vs. replication/synchronisation (ongoing copying that keeps systems in step). Real migrations often use replication to shrink the cutover window.

**Interview angle**
- *Q: How would you migrate a live production database with minimal downtime?* — Convert schema; take an initial snapshot load; start CDC from the snapshot's log position to apply ongoing changes; reconcile counts and checksums; run both systems in parallel; then cut over in a short write freeze once lag is near zero, with a tested rollback plan.

**Your experience hook:** Your MongoDB → Oracle ETL used initial load plus change streams for inserts, updates and deletes, and on the KYC/KYB platform you consolidated 2M+ records from several databases with CDC, using UUID surrogate keys to resolve ID collisions.

**Beyond IBM:** Key-collision handling (surrogate keys, source-system prefixes) and reconciliation are where migrations usually fail, not the copying itself. Keep the migration job idempotent so a failed batch can be rerun safely.

---

## Data movement
*Source: [IBM — What Is Data Movement?](https://www.ibm.com/think/topics/data-movement)*

**In one line:** Data movement covers all the ways data is relocated between systems, from bulk batch loads to continuous streams, and the mechanics of how it gets there.

**Key points**
- Every movement follows extract → transport → ingest. The main design question is latency: does the data need to arrive now or can it wait?
- **Batch movement** (hourly, nightly, weekly): reliable and cheap. **ETL** transforms before loading (quality enforced on entry); **ELT** loads raw data and transforms in the warehouse (now dominant in the cloud, allows re-transforming without re-extracting).
- **Continuous movement:** **streaming** (Kafka, Flink, managed services) for sub-second delivery; **CDC** reads database transaction logs and ships only changes, for low-latency sync and replication.
- **Reverse ETL** pushes warehouse outputs (enriched profiles, scores) back into operational tools such as CRMs.
- Use cases: cloud migration, analytics/BI consolidation, real-time event processing, replication for business continuity, SaaS integration.
- Challenges: scaling and legacy compatibility (sometimes solved by not moving data at all, via federation), compounding latency, quality issues spreading downstream, security of data in transit (PII, residency) and lineage across systems.
- Tools: Kafka, Flink, NiFi; Kinesis, Azure Data Factory, Dataflow; Fivetran and Airbyte for managed connectors.

**Compare / don't confuse with:** Data movement (the mechanics of relocation) vs. data exchange (transfer between stakeholders/organisations) vs. data in motion (the state of data while travelling).

**Interview angle**
- *Q: Why prefer log-based CDC over polling with an `updated_at` column?* — Log-based CDC captures every change including hard deletes and intermediate updates, adds little load on the source and preserves order. Polling misses deletes, depends on the column being maintained and repeatedly scans the table.

**Your experience hook:** You have used CDC for consolidation and Redpanda streams for analytics, and you know the operational side: CDC streams that stall silently need resume tokens and lag monitoring.

**Beyond IBM:** CDC consumers must be idempotent (apply changes as upserts/deletes keyed by primary key, ordered by log position) because connectors deliver at-least-once and may replay after restarts.

---

## Streaming data
*Source: [IBM — What Is Streaming Data?](https://www.ibm.com/think/topics/streaming-data)*

**In one line:** Streaming data is an unbounded, continuous flow of events processed as it arrives rather than in scheduled batches.

**Key points**
- Traits: unbounded, heterogeneous, high velocity, high volume, each record a timestamped event, time-sensitive (value decays) and imperfect (gaps, errors, **out-of-order events**).
- Examples: market data, IoT sensors, social feeds, clickstreams, logs and telemetry, GPS tracking.
- Versus batch: different processing model (continuous vs periodic), infrastructure (stream platforms vs warehouses) and performance needs (always-on, fault-tolerant, low-latency). Many organisations use both.
- Four stages: **ingestion** (Kafka buffers incoming streams), **stream processing** (Flink: filter, enrich, aggregate in motion), **analysis** (live dashboards, automated actions, ML) and **storage** (lakes/lakehouses for raw, warehouses for analytics).
- AI link: fresh features, online learning, continuous retraining, agents reacting in real time.
- Tools: Kafka, Flink (stateful, complex event processing), Spark and Spark Structured Streaming (micro-batch); managed Kinesis, Azure Stream Analytics, Dataflow, Confluent.
- Challenges: scaling, latency vs throughput vs cost, new skills, fault tolerance (loss or duplicates), monitoring lag, governing PII (GDPR, CCPA).

**Compare / don't confuse with:** Batch (bounded input, results after the run) vs. streaming (unbounded input, results continuously). Micro-batching (Spark) sits between them.

**Interview angle**
- *Q: How do you compute a correct 5-minute count when events arrive late or out of order?* — Window by event time, not arrival time; use a watermark that says how late data may be; update or emit corrections for late events within allowed lateness and send anything later to a side output or DLQ.
- *Q: What does "exactly-once" really mean?* — Each event affects the output once, even if it is delivered more than once. It is achieved with checkpointed state plus transactional or idempotent sinks, not by the network never redelivering.

**Your experience hook:** Your current Airflow + Redpanda pipeline and the KYC/KYB queues with idempotency, retries and dead-letter queues give you concrete answers on duplicates and poison messages.

**Beyond IBM:** IBM groups Kafka with "stream processing frameworks"; strictly, Kafka is a durable distributed log for transport and storage, and processing happens in Kafka Streams, Flink or Spark. Also know backpressure (slow consumers) and consumer lag as the key health metric.

---

## Data streaming platform
*Source: [IBM — What Is a Data Streaming Platform?](https://www.ibm.com/think/topics/data-streaming-platform)*

**In one line:** A data streaming platform (DSP) is the infrastructure that continuously captures, processes, stores and serves event streams in real time.

**Key points**
- Shift from warehouse-centric batch ETL to **event-driven architectures**, where events are published, captured, processed and stored as they happen.
- Four qualities: **high throughput**, **low latency**, **fault tolerance**, **scalability**.
- Four layers: **source and ingestion** (APIs, message queues, CDC connectors; in Kafka, producers publish to topics hosted by brokers), **processing** (Flink, Spark Structured Streaming; complex event processing, stateful processing, some batch support), **destination and serving** (apps, dashboards, lakes, lakehouses, warehouses) and **governance** across all layers (lineage, data contracts, schema management, quality rules, security, monitoring).
- AI role: fresh data for predictive models, RAG and agents; streaming feature computation and online inference; online feature stores.
- Benefits: faster insights, time to market, customer service, shared up-to-date data, **replayability** for audits, risk monitoring, lower engineering load.
- Tech: Kafka, Flink, Spark, Storm; Confluent (Kafka + Flink with schema registry, lineage and many connectors); Kinesis, Dataflow, Azure Stream Analytics.
- Use cases: AI, IoT, keeping legacy and new systems in sync during modernisation, data sharing between microservices.

**Compare / don't confuse with:** A message queue such as RabbitMQ (messages are removed once acknowledged; good for task distribution) vs. a log-based streaming platform such as Kafka/Redpanda (events are retained and can be replayed by many independent consumer groups).

**Interview angle**
- *Q: How does Kafka scale and keep order?* — Topics are split into partitions spread across brokers; order is guaranteed only within a partition, so you key messages (e.g. by customer ID) to keep related events together. Consumers in a group share partitions, and replication across brokers gives fault tolerance.
- *Q: Why does replayability matter?* — You can rebuild a downstream table, add a new consumer or backfill after a bug by resetting offsets, without asking the source to resend.

**Your experience hook:** You use both models: RabbitMQ with DLQs on the KYC/KYB platform and Redpanda (Kafka-API compatible) for the current ETL, so you can explain when each fits.

**Beyond IBM:** A schema registry with compatibility rules (backward/forward) is the practical form of "data contracts" on a stream. Replay is limited by retention settings; for long history, use compacted topics or tiered storage, or keep a copy in the lake.

---

## Data latency
*Source: [IBM — What Is Data Latency?](https://www.ibm.com/think/topics/data-latency)*

**In one line:** Data latency is the delay between when data is generated (or requested) and when it is available and ready to use.

**Key points**
- High latency means stale data, which hurts fraud detection, patient monitoring, IoT anomaly detection, e-commerce stock and pricing, and AI agents acting on old inputs.
- "Acceptable" depends on the use case: milliseconds to seconds for fraud or automated decisions; minutes to days for strategic reporting.
- Measure by stamping data at each stage: **event time**, **ingestion time**, **processing time**, **availability time**. From these, derive end-to-end, ingestion, processing and serving latency. Watch clock skew and out-of-order events; use watermarks and synchronised clocks.
- **Network latency** is how fast bytes travel; **data latency** is how fast data becomes usable. A fast network can still have slow data if processing is slow.
- Causes: batch intervals, slow transformations, too many hops between systems; slow storage, limited bandwidth, unindexed or contended databases; legacy batch-oriented design, heavy workloads (including LLM inference), silos.
- Remedies: streaming pipelines and event platforms, incremental processing, faster storage and caching (SSDs, in-memory stores such as Redis), indexing and query tuning, edge processing, zero-copy/lakehouse sharing, fewer silos.

**Compare / don't confuse with:** Latency (delay for one piece of data) vs. throughput (volume processed per unit time). Tuning for one often costs the other: bigger batches raise throughput but add latency.

**Interview angle**
- *Q: How would you set and monitor a freshness SLA?* — Define it per dataset from the consumer's need (e.g. "transactions available within 5 minutes, 99% of the time"), record event and load timestamps, alert on `now - max(event_time)` and on consumer lag, and track percentiles, not averages.

**Your experience hook:** You tuned PostgreSQL to handle ~900 req/s and used Redis queues and caching, and you monitor pipelines with Prometheus and Grafana, which is where latency percentiles and lag alerts live.

**Beyond IBM:** Distinguish *freshness* (how old the newest available data is) from *latency* (per-event delay). Batch pipelines are judged mainly on freshness against an SLA; streaming pipelines on p95/p99 end-to-end latency and consumer lag.

---

## Self-test

1. Why should you verify a checksum after a file transfer, and how do you avoid reading a half-written file?
<details><summary>Answer</summary>A checksum or hash proves the received file matches the original and wasn't corrupted. To avoid partial reads, wait for a completion marker file or an atomic rename before processing.</details>

2. How do you make loading files from a drop folder idempotent?
<details><summary>Answer</summary>Record each loaded file's name and hash in a manifest/control table and skip files already loaded; write to targets by upsert or partition overwrite so a rerun gives the same result.</details>

3. What is the difference between SFTP and FTPS?
<details><summary>Answer</summary>SFTP is the SSH File Transfer Protocol, a separate protocol running over SSH. FTPS is classic FTP secured with TLS.</details>

4. Describe a low-downtime database migration approach.
<details><summary>Answer</summary>Convert schema, take an initial snapshot, start CDC from the snapshot's log position, reconcile counts and checksums, run both systems in parallel, then cut over during a short write freeze once replication lag is near zero, with a rollback plan.</details>

5. Online vs offline migration: when does each fit?
<details><summary>Answer</summary>Online (network transfer) fits when bandwidth is sufficient and speed matters. Offline (shipping a storage appliance) fits very large volumes or constrained bandwidth when the deadline allows shipping time.</details>

6. What does reverse ETL do?
<details><summary>Answer</summary>It moves data from the warehouse back into operational tools (e.g. CRM, marketing platforms) so analytical outputs like scores or enriched profiles drive business actions.</details>

7. Why is log-based CDC usually better than polling an `updated_at` column?
<details><summary>Answer</summary>It captures every change including hard deletes, keeps order, and puts little load on the source. Polling misses deletes and intermediate updates and relies on the column being correctly maintained.</details>

8. How do you handle late and out-of-order events in a windowed aggregation?
<details><summary>Answer</summary>Use event-time windows with a watermark defining allowed lateness; update results for late events within that bound and route anything later to a side output or dead-letter queue.</details>

9. Contrast RabbitMQ and Kafka/Redpanda.
<details><summary>Answer</summary>RabbitMQ is a message queue: messages are removed after acknowledgement, suiting task distribution. Kafka/Redpanda are durable logs: events are retained, ordered per partition and replayable by many independent consumer groups.</details>

10. What is the difference between network latency and data latency, and between latency and freshness?
<details><summary>Answer</summary>Network latency is how fast bytes travel; data latency is how long until data is usable, including processing and serving. Latency is per-event delay; freshness is how old the newest available data is, the usual SLA for batch datasets.</details>
