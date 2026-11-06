# Questions from My Last Interview, With Model Answers

Questions I was actually asked, with answers built from my own experience. Fill the `[brackets]` with real details. Never improvise numbers.

---

## Q1. "How do you make sure the data you fetch is real-time?"
1. **Define it:** agree on a freshness SLA, e.g. "a change reaches the destination within 5 s for 99% of events."
2. **Design:** log-based CDC (reads the WAL/binlog/oplog at commit, no polling) → Redpanda → consumers that process continuously and are scaled to peak load.
3. **Prove it:**
   - End-to-end latency = destination apply time − **source commit timestamp**, tracked as p95/p99 and alerted against the SLA.
   - Consumer lag, both in offsets and in time.
   - Source replication position (Postgres replication slot lag, MongoDB resume token age).
   - **Heartbeat events**, so a quiet table can't hide a dead pipeline.
   - **Reconciliation** of counts and sums, because fast but incomplete data is just wrong data.
4. **Experience:** the KYC CDC stream that stalled silently → resume tokens + alerts on time since the last event.

---

## Q2. "What is MDM? How did you use it?"

### What it is (say this in about 20 seconds)
> "Master data management is how you create and maintain **one trusted, consistent version of your core business entities**, such as customers, businesses, products and locations, across all the systems that use them. It covers the master records, the reference data they depend on, the rules for matching and merging duplicates, and the governance of who can change what."

### Key terms to use
- **Master data:** the core entities that many systems share. For me: **customers** (individuals) and **businesses**.
- **Reference data:** small, controlled code lists that classify other data. For me: **region, woreda, business sector, line of business**.
- **Golden record:** the single best version of an entity after matching and merging duplicates.
- **Match and merge:** finding records that are the same real-world entity (record linkage), then deciding which value of each attribute survives (*survivorship rules*: most recent, most trusted source, most complete).
- **MDM styles:**
  - *Registry:* keeps only a cross-reference index between source systems.
  - *Consolidation:* builds golden records for analytics; the sources are unchanged.
  - *Coexistence:* golden records are also synced back to the sources.
  - *Centralised:* the MDM hub becomes the system of record.
- **Stewardship:** named people own the master data and approve changes.

### How I used it (my answer)
> "On the KYC/KYB onboarding platform, the same customer or business came from several source systems with inconsistent data, so matching and reporting failed. I treated it as an MDM problem with three parts:
> 1. **Reference data:** locations and business classifications were free text ('Addis Ababa', 'A.A', 'Addis'). I standardised on **coded reference lists** (region, woreda, business sector, line of business) and mapped incoming values to codes once. After that, everything matches on codes instead of names.
> 2. **Match and merge:** for people and businesses there's no clean shared code, so I used **record linkage** with Splink. Blocking cut 14.7 billion candidate pairs to 77 million, and the model scored each pair as match or non-match.
> 3. **Golden record and keys:** each resolved entity got a **UUID surrogate key**, which also fixed the collisions we had on source IDs and phone numbers. Downstream systems, including the lending platform, reference that key.
>
> This was a [consolidation / coexistence] style: [the golden record lived in the platform and source systems kept their own copies / was pushed back to ...]."

### Follow-ups to prepare
- *How do you choose the match threshold?* Precision vs. recall. Pairs above a high threshold merge automatically, a middle band goes to **manual review**, everything below stays separate.
- *What if two records were merged wrongly?* Keep lineage (which source records formed the golden record) so you can **unmerge**.
- *Which value wins when sources disagree?* Survivorship rules: [the most recent verified KYC document / the most trusted source / ...].
- *Who maintains the code lists?* A data steward. The lists are versioned and changes are approved. Unmapped values go to a review queue, never a silent NULL.
- *MDM vs. a data warehouse?* MDM produces the trusted *entities*. The warehouse stores the *facts and history* about them and uses MDM keys as its dimension keys.

⚠️ **Backups and disaster recovery are not MDM.** Keep them separate in your answer.

---

## Q3. "How did you use CDC pipelines? How do you manage them?"

### How I used them
> "Two places. On **MICHU**, I first built batch ETL from the source databases into destination databases for BI and predictive analysis. Then I replaced it with **CDC event streaming**, because batch data was [hours] old, batch extraction loaded production, and `updated_at` queries missed deletes. On the **KYC/KYB platform**, CDC consolidated 2M+ records from several source databases into the platform. In a side project I also built a MongoDB → Oracle pipeline on **change streams** (inserts, updates and deletes) with initial-load and full-refresh modes."

**Flow:** source database transaction log ([WAL / binlog / oplog]) → [Debezium / change streams / custom connector] → **Redpanda** topics, keyed by primary key → consumers → destination (upsert/delete).

### How I manage them (lifecycle: answer in this order)
1. **Initial load → streaming handoff.** Record the log position *before* the snapshot, load the snapshot, then stream from that position. Overlapping events are harmless because consumers **upsert**.
2. **Correctness in the consumers:**
   - **Idempotent upserts** on the primary key, so at-least-once delivery doesn't create duplicates.
   - **Ordering per key:** partition by primary key so one record's changes stay in order. Optionally skip events older than the version already stored.
   - **Deletes:** delete events or tombstones become hard deletes or a soft-delete flag.
   - **Retries with backoff** for transient errors, and a **dead-letter queue** for bad events, with a replay process.
3. **Restarts and positions.** Consumers commit offsets *after* writing. Source connectors keep their **resume token / LSN**. If downtime exceeds log retention, the position is gone, so you need a **re-snapshot** procedure.
4. **Schema changes:** additive changes are fine. Breaking changes are agreed with the source team (data contract, schema registry). Validation at the consumer sends unknown shapes to the DLQ instead of crashing.
5. **Monitoring and alerts:**
   - Consumer lag, in offsets and in time.
   - End-to-end latency p95.
   - Time since the last event, plus heartbeats.
   - DLQ size.
   - Connector status.
   - On Postgres, **replication slot size**: a stuck slot makes the source disk fill up with WAL.
6. **Reconciliation:** scheduled count and checksum comparisons between source and destination, to catch silent data loss.
7. **Operations:** runbooks for restart, re-snapshot and DLQ replay. Dashboards in Grafana. Logs in ELK for tracing a single record.

**The incident to tell:** "A CDC stream once **stalled silently**: no errors, the data just stopped. [How it was noticed]. Root cause: [...]. Fix: **resume tokens**, so consumers continue exactly where they stopped, plus alerts on lag and time since the last event. Now a stall is detected in minutes."

---

## Q4. "Design a data pipeline"

### Use this framework every time (say the steps out loud)
1. **Clarify requirements (2–3 min):**
   - **Sources:** which systems, what kind (databases, APIs, files, events)?
   - **Volume and growth:** rows or GB per day.
   - **Freshness:** real-time, hourly or daily?
   - **Consumers:** BI, ML, applications, regulators.
   - **Correctness needs:** in finance, no duplicates and no losses.
   - **PII and compliance**, **team size**, **cloud and budget**.
2. **High-level architecture:** sources → ingestion → storage layers → transformation → serving, with orchestration, quality checks, monitoring and governance running across all of it.
3. **Walk through each stage and give a reason for every choice.**
4. **Failure handling:** retries, DLQ, idempotency, backfills, late data, schema changes.
5. **Operations:** monitoring, alerts, data quality, cost, ownership.
6. **Trade-offs, and how the design would change at 10× scale.**

### Worked example: "Design a pipeline for loan data from the lending platform for BI and credit-risk models"
**Clarify:** 3 Postgres databases plus 1 MongoDB, ~500 GB, +20 GB/month. Dashboards need data within 5 minutes. Models train daily. PII is involved.

```
Postgres x3 ─┐  log-based CDC
MongoDB  ────┼──────────────► Redpanda (topic per table, key = primary key)
APIs/files ──┘ batch (Airflow)        │
                                       ▼
                       Bronze: raw change events, append-only (object storage / Iceberg)
                                       │  dedup, upsert by key, type, PII pseudonymised
                                       ▼
                       Silver: current state + history (SCD2), MDM keys, reference codes
                                       │  dbt models, tests
                                       ▼
                       Gold: facts and dimensions (loans, repayments, customers), ML features
                                       │
                         BI dashboards · credit-risk models · reverse-ETL to apps

Across all layers: Airflow orchestration · dbt tests + freshness · Prometheus/Grafana (lag, latency, DLQ)
                   · IAM / row-level security per bank · Terraform · CI/CD
```

**Why each choice:**
- **CDC** for the databases: near real-time, captures deletes, no query load on production.
- **Batch** for APIs and files: they don't produce change logs, and daily is enough.
- **Bronze/silver/gold:** you can always replay from raw data, and the layers make cleaning and business logic easy to test.
- **Pseudonymise PII early**, in silver, so only a few roles ever see raw identities.
- **dbt** for transformations: version control, tests, lineage, documentation.
- **Idempotent** everywhere: upserts, partition overwrite, so retries and backfills are safe.

**Failure handling:** retries with backoff. DLQ with replay. Lookback windows for late data. Schema changes validated at ingestion. Backfills run by date partition.

**Operations:** freshness SLA alerts, row-count reconciliation, DLQ size alerts, cost per pipeline, a runbook per failure type.

**At 10× scale:** more partitions and consumers, compaction of small files, incremental models instead of full rebuilds, possibly Flink for real-time aggregates.

---

## Q5. Data lake vs. data warehouse vs. lakehouse questions

### Definitions (one sentence each)
- **Data warehouse:** a managed, structured store optimised for SQL analytics and BI, with a schema defined up front (*schema-on-write*). Examples: BigQuery, Snowflake, Redshift.
- **Data lake:** cheap object storage holding raw data of any type in open files (Parquet, JSON, images), with the schema applied when the data is read (*schema-on-read*). Examples: S3, ADLS, GCS.
- **Data lakehouse:** a data lake plus an **open table format** (Iceberg, Delta, Hudi), which adds warehouse features (ACID transactions, updates and deletes, schema enforcement, time travel) on cheap storage that **many engines** can use.

### Comparison
| | Warehouse | Lake | Lakehouse |
|---|---|---|---|
| Data | Structured | Anything, raw | Anything, with tables on top |
| Schema | On write | On read | Enforced by the table format, evolvable |
| Transactions / updates / deletes | Yes | No | Yes (MERGE, DELETE) |
| Cost | Higher | Lowest | Low storage, compute of your choice |
| Engines | Vendor's | Any, but no guarantees | Many (Spark, Trino, Flink, DuckDB, Snowflake, BigQuery) |
| Best for | BI, SQL analysts | Cheap raw storage, ML experiments | BI + ML + streaming on one copy of the data |
| Main risk | Cost, lock-in | "Data swamp": no quality or governance | Operations: small files, compaction, catalog |

### Questions they may ask, with short answers
- **"Why did lakehouses appear?"** Lakes were cheap but unreliable: no transactions, no updates, swamp problems. Warehouses were reliable but expensive and closed. Table formats added a transaction log on top of lake files, giving the reliability of a warehouse at the cost of a lake.
- **"How does a lakehouse give ACID on files?"** Files are never modified in place. Every write creates new files plus a new **snapshot** in the metadata, and the commit atomically swaps the pointer to the current snapshot. Readers see either the old or the new version, never half of each.
- **"Schema-on-write vs. schema-on-read?"** On write: data is validated when it's loaded, so bad data is rejected early and queries are fast. On read: anything can be loaded and structure is applied when queried, which is flexible but pushes quality problems onto consumers.
- **"Where does CDC data go in a lakehouse?"** Raw change events are appended to bronze, then **MERGE**d into silver tables. The table format makes the upserts and deletes possible, which a plain lake can't do.
- **"How do you handle a GDPR deletion in a lakehouse?"** `DELETE` the rows, then expire old snapshots and vacuum the files. Until then, time travel can still read the deleted data.
- **"What's a data mart?"** A subject-specific subset of the warehouse (e.g. lending, collections) for one team.
- **"What's the medallion architecture?"** Bronze (raw) → silver (cleaned, conformed) → gold (business-ready).
- **"Which would you choose for us?"** Don't answer immediately. Ask about volume, data types, the team, ML needs and lock-in concerns. Then: *"For a small team with mostly structured data, I'd start with a managed warehouse. I'd move to a lakehouse when volume, ML on raw data, multiple engines or cost justify it. Either way, I'd keep raw data in open formats so we're not locked in."*

### Tie it to your experience
> "Our CDC pipelines fed destination databases for BI and predictive analysis, which was effectively a warehouse pattern. If I designed it today at larger scale, I'd land the CDC events in an Iceberg bronze layer and MERGE them into silver. That gives us upserts and deletes from CDC, time travel to reproduce the exact dataset a credit model was trained on, and cheap storage, while BI still queries gold tables."
