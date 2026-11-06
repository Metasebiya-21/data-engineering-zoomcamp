# 4. Data platforms

> This section covers the places analytical data lives and the layers around them: the data platform as a whole, warehouses, marts, lakes, the open table formats (Delta Lake, Apache Iceberg) that turned lakes into lakehouses, the lakehouse itself, feature stores for ML, and master data management for core business entities. In interviews this is where design questions land: "where would you put this data, in what format, and why?" Expect to defend trade-offs between cost, performance, governance and flexibility, and to explain what a table format actually does (ACID commits, time travel, schema evolution, MERGE/deletes, compaction) rather than repeating "lakehouse = lake + warehouse".

## Data platform
*Source: [IBM — What Is a Data Platform?](https://www.ibm.com/think/topics/data-platform)*

**In one line:** A data platform (or "data stack") is the combined set of tools and infrastructure that collects, stores, transforms, serves, governs and monitors an organisation's data.

**Key points**
- IBM names five core layers: storage and processing, ingestion, transformation, BI/analytics, and data observability. Common extra layers are data discovery, governance, cataloguing/metadata, and ML/AI.
- Types: enterprise data platform (central, historically on-premises; modern versions rest on availability through separated storage and compute, and elasticity through autoscaling compute), big data platform (distributed, high-speed processing of huge volumes), cloud data platform (pay-as-you-go, no hardware to run) and customer data platform (a unified 360-degree view of each customer).
- Storage options are warehouse, lake or lakehouse.
- Ingestion is batch (cheaper, the most common) or streaming (fresher, costlier because sources are watched continuously).
- Transformation happens before load (ETL, typical for on-premises warehouses) or after load (ELT, enabled by elastic cloud warehouses).
- Observability means tracking, logging, alerting and anomaly detection, so teams can see where data is slow, late or broken.
- The goal is to break down silos and give the organisation a single source of truth plus self-service access.

**Compare / don't confuse with:** Customer data platform vs. cloud data platform: both are abbreviated "CDP". One is a business application for unifying customer profiles; the other is cloud infrastructure for data in general.

**Interview angle**
- *Q: Walk me through the layers of a data platform you'd build for a small company.* — Ingestion (CDC or batch extracts plus a stream where freshness matters), cheap object or warehouse storage, transformation in SQL/dbt orchestrated by Airflow, a BI layer, and observability from day one: freshness, volume and schema checks with alerting. Add governance (access control, PII handling) and a catalog as the team grows.

**Your experience hook:** My current Airflow + Redpanda pipelines feeding analysis-ready and ML datasets, monitored with Prometheus and Grafana, cover the ingestion, transformation and observability layers end to end.

**Beyond IBM:** IBM lists observability last, but practitioners treat it as a requirement from the start, because silent failures (a stalled CDC stream, a half-loaded partition) are the costliest kind.

---

## Data warehouse
*Source: [IBM — What Is a Data Warehouse?](https://www.ibm.com/think/topics/data-warehouse)*

**In one line:** A data warehouse is a central, query-optimised store that integrates cleaned data from many source systems for BI and analytics.

**Key points**
- Classic three tiers: the bottom tier is the database server loaded by ETL/ELT, the middle tier is an OLAP engine, and the top tier is reporting, dashboards and ad hoc query tools.
- OLAP comes in three variants: MOLAP (pre-built cubes, fastest), ROLAP (works directly on relational tables) and HOLAP (a mix). OLTP captures transactions; OLAP analyses data that has already been captured.
- Deployment can be on-premises (MPP or SMP appliances, chosen for strict compliance), cloud (managed SaaS, elastic, pay-as-you-go) or hybrid.
- Schemas are dimensional: star (one fact table plus denormalised dimensions, fast and simple), snowflake (normalised dimensions, less redundancy, more joins) and galaxy/fact constellation (several fact tables sharing dimensions).
- Components: ETL/ELT tools, an API layer, the central database, metadata, a sandbox for experiments, and access tools such as Tableau, Looker or Qlik.
- Types: the enterprise data warehouse (EDW), the operational data store (ODS: a frequently refreshed snapshot of current operational data) and the data mart.
- To control cost, separate storage from compute and use open formats. For quality, "shift left": catch problems close to the source.

**Compare / don't confuse with:** Database vs. warehouse: an OLTP database serves one application with fast row-level writes; a warehouse integrates many sources for scans and aggregations. ODS vs. EDW: an ODS holds current state that is refreshed often; an EDW holds integrated history.

**Interview angle**
- *Q: Star or snowflake schema in a modern cloud warehouse?* — Usually star. Storage is cheap and columnar engines handle wide denormalised dimensions well, so you save joins and keep the model easy for analysts. Snowflake a dimension only when it is huge or shared and changes independently.

**Your experience hook:** In the Zoomcamp I built partitioned and clustered BigQuery tables; in my MongoDB → Oracle incremental ETL, Oracle played the warehouse-style target for change-stream inserts, updates and deletes.

**Beyond IBM:** Physical OLAP cubes are now mostly legacy. Cloud warehouses (BigQuery, Snowflake, Redshift) run ROLAP-style SQL on columnar, MPP storage, with partitioning, clustering and materialised views doing the job cubes used to do.

---

## Data mart
*Source: [IBM — What Is a Data Mart?](https://www.ibm.com/think/topics/data-mart)*

**In one line:** A data mart is a subject-focused slice of warehouse (or other) data built for one department or business area.

**Key points**
- Purpose: give a team such as finance, sales or marketing quick access to the data it needs without navigating the whole warehouse.
- Benefits: lower cost, simpler access, faster insight, easier upkeep and faster build than a full warehouse (IBM cites a rough size under 100 GB).
- **Dependent** marts are carved from an EDW (top-down). **Independent** marts are standalone and pull straight from sources, which risks silos. **Hybrid** marts mix warehouse data with other sources.
- Usually modelled dimensionally. IBM lists star, snowflake and "vault" structures.
- Design steps: gather requirements, identify sources, pick the data subset, then design a logical schema that lines up with the warehouse.
- Cloud warehouses that separate storage and compute make it cheap to host many marts in one place.

**Compare / don't confuse with:** Mart vs. warehouse: same technology, different scope (one domain vs. the whole enterprise). A dependent mart shares the warehouse's conformed definitions; an independent mart can drift from them.

**Interview angle**
- *Q: What is the risk with independent data marts?* — Each one redefines metrics and entities in its own way, so "revenue" or "active customer" differs by department. Dependent marts built on conformed dimensions avoid this.

**Beyond IBM:** Data Vault (hubs, links, satellites) is really an integration-layer modelling method for the warehouse core, not a mart schema. Marts are usually star schemas built on top of it. In dbt projects, "marts" is simply the final layer of business-facing models.

---

## Data lake
*Source: [IBM — What Is a Data Lake?](https://www.ibm.com/think/topics/data-lake)*

**In one line:** A data lake is low-cost storage, usually cloud object storage, that holds raw structured, semi-structured and unstructured data in its native format, with schema applied on read.

**Key points**
- James Dixon coined the term in 2011. Early lakes ran on Hadoop HDFS on-premises, then moved to object stores such as S3, Azure Blob and GCS.
- Layers: ingestion (mostly ELT, schema-on-read), storage, catalog/metadata, processing (Spark, Hive, Pandas and ML frameworks, with compute separate from storage), security and governance (IAM, encryption, GDPR/HIPAA), and access (SQL engines such as Presto, plus APIs).
- Benefits: flexible ingestion (batch or streaming, any format), low cost, independent scaling of storage and compute, and a central place that reduces silos.
- Use cases: all-purpose and "just-in-case" storage, backup and archive (including regulatory retention), AI/ML training data, and a staging ground for integration.
- Challenges: data quality, because nothing is enforced on write; security, because a large central store is a target and is not secure by default; and query performance as volumes grow.
- Without a catalog and governance, a lake becomes a **data swamp**.

**Compare / don't confuse with:** Schema-on-read (lake) vs. schema-on-write (warehouse): the lake accepts anything and pushes validation to whoever reads the data; the warehouse rejects or fixes data on the way in.

**Interview angle**
- *Q: How do you stop a lake becoming a swamp?* — Catalog every dataset with an owner and schema, separate raw from curated zones (bronze/silver/gold), enforce schemas with a table format on the curated zones, add quality checks and access controls, and expire data you don't need.

**Your experience hook:** In the Zoomcamp I used GCS as the raw landing lake (Parquet files) in front of BigQuery.

**Beyond IBM:** A plain file lake has no atomic multi-file commits. Readers can see half-written output, updates and deletes mean rewriting files by hand, and many small files plus slow directory listings hurt performance. Open table formats were created to fix exactly these problems.

---

## Delta Lake
*Source: [IBM — What is Delta Lake?](https://www.ibm.com/think/topics/delta-lake)*

**In one line:** Delta Lake is an open table format that adds a transaction log on top of Parquet files, giving a data lake ACID commits, versioning and schema control.

**Key points**
- Databricks built it in 2016 and open-sourced it in 2019. Delta is the "how" (table format); the lake is the "where" (storage).
- It has two parts: Parquet data files, and the transaction log (`_delta_log`): JSON commit files, periodically rolled up into Parquet checkpoints. The log records which files belong to each version, along with schema and per-file statistics such as min/max values.
- Parquet files are immutable, so changes are *logical*. A commit adds and removes files in the log. A dropped column can be hidden in metadata rather than rewritten.
- Features: ACID transactions, schema **enforcement** (bad writes rejected) and **evolution** (add columns, change types), data skipping using min/max statistics, Z-ordering to co-locate related data, UPDATE/DELETE/MERGE, version history with **time travel**, batch and streaming reads and writes, and connectors for Spark, Flink, Trino and Hive.
- It has no native security model. Governance tools read the log for audit and RBAC.
- Alternatives: Iceberg (Parquet/ORC/Avro, three-level metadata tree) and Hudi (a "timeline", built for incremental upserts and CDC).

**Compare / don't confuse with:** Delta Lake (format) vs. data lake (storage) vs. Databricks (vendor platform). Also time travel (querying an old version) vs. restore/rollback (making an old version current again). IBM blurs these two.

**Interview angle**
- *Q: How does Delta give ACID on object storage?* — Each write produces new Parquet files, then tries to commit by writing the next numbered log entry. Only one writer can create a given version number, so commits are atomic and readers see only committed snapshots. Concurrent writers use optimistic concurrency: on conflict they check for overlap and retry or fail.
- *Q: How would you apply CDC (including deletes) to a Delta table?* — Deduplicate each batch to the latest change per key, then `MERGE INTO` on the key with `WHEN MATCHED AND op='d' THEN DELETE`, `WHEN MATCHED THEN UPDATE` and `WHEN NOT MATCHED THEN INSERT`. Keyed merges are idempotent on replay.

**Your experience hook:** My MongoDB → Oracle change-stream ETL already applies inserts, updates and deletes by key, which is the same upsert/delete pattern MERGE expresses on a Delta table.

**Beyond IBM:** Deletes are logical until `VACUUM` removes unreferenced files (default retention 7 days). That matters for GDPR erasure, and it also limits how far back time travel can go. Frequent small writes need compaction (`OPTIMIZE`). Deletion vectors (a merge-on-read option) speed up deletes and updates. Delta 4.0, described by IBM as "planned", has since been released.

---

## Apache Iceberg
*Source: [IBM — What Is Apache Iceberg?](https://www.ibm.com/think/topics/apache-iceberg)*

**In one line:** Apache Iceberg is an engine-neutral open table format that tracks a table as a tree of metadata files and snapshots, so many engines can safely read and write the same files with ACID guarantees.

**Key points**
- Netflix and Apple created it in 2017 to fix Hive tables, which had no atomic commits and tracked data by directory. It became a top-level Apache project in 2020.
- Three layers: the **catalog** (maps a table name to its current metadata file; the pointer swap is atomic; Hive Metastore, AWS Glue or REST catalogs), the **metadata layer** (metadata file → manifest list → manifest files) and the **data layer** (Parquet, ORC or Avro files).
- The metadata file holds the schema, partition spec and the list of snapshots. Each snapshot has a manifest list. Manifests track individual data files with statistics, so tables are tracked by *file*, not by directory.
- **Hidden partitioning**: partitions are derived from column transforms (e.g. day of a timestamp), so queries filter on the real column and users never handle partition columns. **Partition evolution**: change the spec without rewriting old data; old files keep the old layout.
- **Schema evolution** without rewrites (add, drop, rename, reorder). **Time travel** and rollback through snapshots.
- Maintenance features: compaction of small files and Bloom filters for read skipping.
- Engines: Spark, Trino/Presto, Flink, Hive, Impala and others.

**Compare / don't confuse with:** see the table below.

| | Delta Lake | Apache Iceberg | Apache Hudi |
|---|---|---|---|
| Origin | Databricks, 2016 | Netflix/Apple, 2017 | Uber, 2016 |
| Metadata | Ordered transaction log + checkpoints | Catalog → metadata file → manifest list → manifests | Timeline of actions |
| Data files | Parquet | Parquet, ORC, Avro | Parquet, ORC or HFile base files; Avro log files |
| Strength | Tight Spark/Databricks integration | Engine-neutral; hidden partitioning and partition evolution | Record-level upserts, incremental pulls, CDC |
| Row changes | Copy-on-write; deletion vectors | Copy-on-write or merge-on-read (delete files) | Copy-on-write or merge-on-read tables |

**Interview angle**
- *Q: Why do column IDs matter for schema evolution?* — Iceberg tracks columns by ID, not by name or position, so renaming or reordering a column never makes old files return the wrong data, and a dropped-then-re-added column cannot resurrect old values.
- *Q: Copy-on-write vs. merge-on-read?* — Copy-on-write rewrites affected files on every update: slower writes, fastest reads. Merge-on-read writes small delete files and merges them at read time: cheap frequent writes (good for CDC), but you need regular compaction.

**Beyond IBM:** IBM's performance claims rest on one 2022 benchmark. Results depend heavily on versions and configuration, so don't quote it as settled. Production Iceberg needs routine maintenance: `rewrite_data_files` (compaction), `expire_snapshots` (this also bounds time travel and physically completes deletes) and `remove_orphan_files`. The catalog choice is a real architecture decision.

---

## AWS Lake Formation
*Source: [IBM — What Is AWS Lake Formation?](https://www.ibm.com/think/topics/aws-data-lake-formation)*

**In one line:** AWS Lake Formation is a managed AWS service for building an S3-based data lake quickly and centrally governing who can see which data in it.

**Key points**
- The article is short and partly promotional. It argues lakes are popular because they are flexible, cheap and free of proprietary formats, which makes security more important as more people need access.
- Three stated capabilities: build a lake fast (move, store, catalog and organise data), manage security centrally (define and enforce access, governance and audit policies in one place) and make data easy to discover and share.
- It integrates with S3. You still need a query engine; the article suggests Presto, giving the stack S3 + Presto + Lake Formation.

**Compare / don't confuse with:** Lake Formation (permissions and governance over the lake) vs. the Glue Data Catalog (the metadata store it builds on) vs. S3 (the storage itself).

**Interview angle**
- *Q: Why not just use S3 bucket policies for lake security?* — Bucket and IAM policies work at object or prefix level. Lake Formation grants table-, column- and row-level permissions on catalog tables, enforced consistently across engines such as Athena, Redshift Spectrum and EMR.

**Beyond IBM:** In practice Lake Formation is used for fine-grained, tag-based access control (LF-Tags) and cross-account data sharing on top of the Glue Data Catalog. The comparable controls elsewhere are Unity Catalog on Databricks and BigLake/Dataplex on GCP.

---

## Data lakehouse
*Source: [IBM — What Is a Data Lakehouse?](https://www.ibm.com/think/topics/data-lakehouse)*

**In one line:** A data lakehouse puts warehouse-style management (transactions, schemas, governance, fast SQL) directly on top of cheap open files in object storage, so one copy of the data serves both BI and ML.

**Key points**
- The problem it solves: the old two-tier pattern (lake, then ETL into a warehouse) duplicated data, added staleness and created extra places for errors.
- It works through open table formats (Delta, Iceberg, Hudi) acting as a metadata layer over Parquet/ORC files, which enables ACID, time travel, versioning, schema evolution and DML.
- Five layers: ingestion (batch or real-time), storage (object storage, columnar files), metadata (unified catalog plus table format), API (engines and ML frameworks reading directly) and consumption (BI, ML jobs).
- **Medallion architecture**: bronze (raw and immutable, so you can always replay), silver (cleaned, conformed, deduplicated) and gold (business-ready aggregates and metrics). Any gold row can be traced back to its source.
- Features: open formats (no lock-in, many engines on one copy), unified data, low-cost storage, workload flexibility, governance, independent scaling of storage and compute, and streaming support.
- Challenges: complex migration, balancing open access with security, and keeping query performance up as data grows. A lakehouse can replace a warehouse, but many organisations run both.

**Warehouse vs. lake vs. lakehouse**

| | Data warehouse | Data lake | Data lakehouse |
|---|---|---|---|
| Data types | Mainly structured | Anything (raw files) | Anything; tables over open files |
| Schema | On write, enforced | On read, not enforced | Enforced on write per table, with evolution |
| Storage | Proprietary managed storage (some now read open formats) | Object storage, open files | Object storage, open files + table format |
| Transactions | Full ACID, multi-table | None (file-level only) | ACID per table via the commit log/catalog |
| Updates/deletes | Native DML/MERGE | Rewrite files by hand | MERGE/UPDATE/DELETE via copy-on-write or merge-on-read |
| Time travel | Some (e.g. BigQuery/Snowflake, limited window) | No | Yes, snapshots/versions until expired or vacuumed |
| Performance | Highest, tuned engine; low latency | Weak without layout work | Good: stats, data skipping, clustering, compaction; needs maintenance |
| Cost | Higher; compute and storage often bundled or premium | Lowest | Low storage; pay for chosen engines |
| Engines | Usually one (the vendor's) | Many | Many on the same copy |
| Best for | Governed BI, finance reporting, high concurrency | Raw landing, archive, ML on files | Mixed BI + ML on one copy at scale, streaming upserts |
| Main risk | Lock-in, cost at scale, poor fit for unstructured data | Swamp, no consistency | Operational overhead (compaction, snapshot expiry, catalog) |

**Interview angle**
- *Q: Would you recommend a lakehouse to a small company?* — Not by default. Under a few terabytes, a managed warehouse (or plain Postgres) is simpler and cheaper to run. A lakehouse pays off when data volume, multiple engines, ML on raw data or vendor-neutrality justify the maintenance work.

**Your experience hook:** My Airflow + Redpanda pipelines that produce analysis-ready and ML datasets follow the same raw → cleaned → curated progression as medallion layers.

**Beyond IBM:** The gap is narrowing from both sides: warehouses now read and write Iceberg (e.g. BigQuery and Snowflake Iceberg tables), and lakehouses add warehouse-like serving engines. The real decision is usually "open format and many engines" vs. "one managed engine".

---

## Feature store
*Source: [IBM — What Is a Feature Store?](https://www.ibm.com/think/topics/feature-store)*

**In one line:** A feature store is a system that defines, stores and serves ML features once, so training and production inference use the same values computed the same way.

**Key points**
- A feature is a model input derived from raw data, e.g. "transactions in the last 7 days" for fraud detection.
- The main problem it solves is **training-serving skew**: features computed with slightly different logic offline and online, which quietly degrades the model.
- Dual storage: the **offline store** (history on a warehouse or lake, used to build training sets) and the **online store** (latest values in a low-latency key-value store such as Redis, used at inference).
- Other parts: ingestion/transformation (batch, streaming or on-demand), a serving API/SDK, a **feature registry** (definitions, lineage, versions, owners) and orchestration (scheduled computation, backfills, recomputing when a definition changes).
- Benefits: reuse instead of re-implementing features, consistency, collaboration, governance, real-time ML and MLOps scale.
- Choosing one: fit with the existing platform, open source (Feast) vs. managed (Tecton, SageMaker Feature Store) vs. building your own (e.g. Uber Michelangelo), latency needs and **point-in-time correctness**.

**Compare / don't confuse with:** Offline store vs. online store: one is for history and training, the other for current values at serving time. A feature store is also not a vector database, which stores embeddings for similarity search.

**Interview angle**
- *Q: What is point-in-time correctness and why does it matter?* — When building a training row for an event at time T, you may join only feature values known before T. Using later values leaks the future, so the model looks great offline and fails in production.

**Your experience hook:** My credit-scoring features built with MongoDB aggregation pipelines (with factor analysis and PCA) are exactly the kind of logic a feature store would centralise, so scoring at decision time uses the same definitions as training.

**Beyond IBM:** Small teams often don't need a dedicated product. Versioned SQL/dbt feature tables plus a Redis cache, with one shared transformation code path, give most of the benefit.

---

## Cloud data lake vs. data warehouse vs. data mart
*Source: [IBM — Cloud Data Lake vs. Data Warehouse vs. Data Mart](https://www.ibm.com/think/topics/cloud-data-lake-vs-data-warehouse-vs-data-mart)*

**In one line:** Lakes, warehouses and marts are complementary cloud repositories: a lake for raw data, a warehouse for integrated structured data, and a mart for a focused departmental subset.

**Key points**
- **Lake**: raw, unaltered data. It is cheap to keep, but analysis needs preparation first (e.g. a city stores traffic and outage data until it has budget to analyse it).
- **Warehouse**: unified, consistent data that is easy to mine and analyse. It can be delivered as a managed cloud service, as self-run software (for control and compliance) or as an appliance.
- **Mart**: smaller, organised for one business unit or purpose, often used for current, actionable analysis. It can be dependent, independent or hybrid, and structured as star, snowflake or vault.
- The examples combine all three: a manufacturer runs a cloud warehouse with dependent marts for finance, manufacturing and marketing; a city mixes a cloud lake, an on-premises warehouse and a hybrid mart.
- Misconceptions IBM rejects: one size fits all; cloud data is stranded; cloud is less secure; cloud is always more expensive. Some regulated cases still need on-premises systems.
- Choose based on goals, budget, volume, access frequency and how soon the data is needed.

**Compare / don't confuse with:** see the warehouse vs. lake vs. lakehouse table in the Data lakehouse entry. A mart is a scope (a subset for one audience), not a different storage technology.

**Interview angle**
- *Q: A team wants its own mart. What do you check first?* — Whether it can be a dependent mart (views or models over the warehouse with shared dimensions), so metric definitions stay consistent. Build an independent copy only if there's a real latency, cost or isolation reason.

**Beyond IBM:** This article is older. It omits the lakehouse and overstates the lake/warehouse cost split now that warehouses separate storage from compute and can read open formats.

---

## Master data management (MDM)
*Source: [IBM — What is Master Data Management?](https://www.ibm.com/think/topics/master-data-management)*

**In one line:** MDM is the discipline and tooling that produces one trusted "golden record" for each core business entity (customer, product, supplier, location) and shares it across systems.

**Key points**
- IBM's six data categories: unstructured, transactional, metadata, hierarchical, reference (country and currency codes) and master (key business entities). Master data changes slowly but is high-value and widely reused.
- Domains: customer (including employee and salesperson), product (part, asset, store), supplier, location, and others such as contract or licence.
- Building it: integrate sources → cleanse and standardise → deduplicate/match → reconcile conflicts → enrich → govern, with audit trails and version history.
- Uses: CRM (no duplicate mailings or wrong billing addresses), product information management, **KYC and regulatory compliance**, ERP consistency, merging data after M&A, analytics and personalisation.
- Challenges: keeping one version of the truth in sync across copies, data quality, reconciling conflicting values, formats and update frequencies, transformation, and balancing normalisation against usability.
- Tool functions: integration, cleansing, standardisation, reconciliation/enrichment, governance, hierarchy management, ERP/CRM connectors, audit/versioning.

**Compare / don't confuse with:** Master data (entities: who and what) vs. reference data (code lists used for classification) vs. transactional data (events involving those entities). Also MDM vs. a customer data platform: MDM governs a golden record for all systems; a CDP is mostly a marketing profile tool.

**Interview angle**
- *Q: How do you build a golden customer record from several source systems?* — Standardise fields (names, phones, addresses), block and then match candidate pairs with deterministic and probabilistic rules, cluster the matches, apply survivorship rules (which source wins for each attribute), assign a stable surrogate master ID, and keep the cross-reference to source IDs for lineage and corrections.

**Your experience hook:** On the KYC/KYB platform I consolidated 2M+ records from several source databases via CDC, using UUID surrogate keys to resolve ID and phone-number collisions, and I've done entity resolution with Splink where blocking cut 14.7B candidate pairs to 77M: this is MDM's match-and-merge core.

**Beyond IBM:** MDM is usually described by implementation style: registry (index of links only), consolidation (golden record built for analytics), coexistence (golden record synced back to sources) and centralised (the hub is the system of entry). Survivorship rules and data stewardship workflows for unclear matches are what make it work in practice.

---

## Self-test

1. What are IBM's five foundational layers of a data platform, and which one do practitioners say must not be an afterthought?
<details><summary>Answer</summary>Storage/processing, ingestion, transformation, BI/analytics and observability. Observability (freshness, volume, schema and anomaly alerts) should exist from day one, because silent pipeline failures are the most damaging.</details>

2. Explain the difference between an ODS, an EDW and a data mart.
<details><summary>Answer</summary>An ODS holds a frequently refreshed snapshot of current operational data for day-to-day decisions. An EDW holds integrated history across the whole enterprise. A data mart is a subject- or department-focused subset, ideally dependent on the EDW's conformed dimensions.</details>

3. Why can a plain Parquet-on-S3 data lake not give you ACID guarantees, and what problems does that cause?
<details><summary>Answer</summary>There is no atomic commit across many files and no record of which files form the current table. Readers can see partial writes, concurrent writers can corrupt results, and updates/deletes mean rewriting files by hand. Small files and directory listings also slow queries.</details>

4. How does Delta Lake make an immutable Parquet table behave as if it were mutable?
<details><summary>Answer</summary>Through the `_delta_log` transaction log. Each commit records which files are added and removed (plus schema and stats), so an update writes new files and logically removes old ones. Readers build the current snapshot from the log. Physical removal happens later with VACUUM.</details>

5. Describe Iceberg's metadata hierarchy from catalog down to data files.
<details><summary>Answer</summary>The catalog points a table name at its current metadata file (atomic pointer swap). The metadata file holds the schema, partition specs and snapshots. Each snapshot points to a manifest list. The manifest list points to manifest files. Manifest files list the data files with column stats.</details>

6. What are hidden partitioning and partition evolution, and why do they matter?
<details><summary>Answer</summary>Hidden partitioning derives partitions from column transforms (e.g. day(ts)), so users filter on real columns and can't get partition filters wrong. Partition evolution changes the spec for new data without rewriting old files, and queries still work across both layouts.</details>

7. Copy-on-write vs. merge-on-read: which suits a high-frequency CDC upsert stream, and what maintenance does it require?
<details><summary>Answer</summary>Merge-on-read: writes are cheap (delete files or deletion vectors) and the merge happens on read. It needs regular compaction to fold deletes into base files, plus snapshot expiry or vacuum to remove old files.</details>

8. A user asks you to delete their data under GDPR from a Delta or Iceberg table. Why is `DELETE` alone not enough?
<details><summary>Answer</summary>DELETE only creates a new version. The old files still exist and are reachable through time travel until you run VACUUM (Delta) or expire_snapshots plus orphan-file cleanup (Iceberg) past the retention window.</details>

9. What is training-serving skew, and how does a feature store prevent it? What is point-in-time correctness?
<details><summary>Answer</summary>Skew is when features are computed differently in training and in production. A feature store defines each feature once and serves it from both an offline store (training) and an online store (inference). Point-in-time correctness means a training row only uses feature values known as of its event time, which avoids leaking future data.</details>

10. In MDM, what are survivorship rules and how does master data differ from reference data?
<details><summary>Answer</summary>Survivorship rules decide which source's value wins for each attribute when matched records disagree (e.g. most recent, most trusted source). Master data describes core entities (customers, products); reference data is a set of classification codes (country and currency codes) used to categorise other data.</details>
