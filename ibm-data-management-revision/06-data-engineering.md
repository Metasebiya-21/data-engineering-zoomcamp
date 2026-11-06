# 6. Data engineering

> This section covers the core work of a data engineer: getting data in (acquisition), moving and shaping it (pipelines, cleaning, validation, enrichment, wrangling, reconciliation), and running all of it reliably without people pressing buttons (automation, DAGs, orchestration). IBM also adds the AI angle: pipelines built for AI workloads and AI agents that build and repair pipelines. In interviews this is the part you will be quizzed on most directly: expect to explain how you design a pipeline, how you make it safe to rerun, how you catch bad data before it reaches users, and how you choose and use an orchestrator.

## Data engineering
*Source: [IBM — What Is Data Engineering?](https://www.ibm.com/think/topics/data-engineering)*

**In one line:** Data engineering is the discipline of designing, building and running the systems that collect, store and prepare data at scale so others can analyse it or train models on it.

**Key points**
- Data engineers turn raw data scattered across operational systems into *core datasets*: purpose-built, reliable datasets for a specific downstream use. IBM names three qualities: easy to use (the "data as a product" idea), contextual (shows change over time, not just a snapshot) and comprehensive (integrates all relevant sources).
- The integration pipeline has three phases: **ingestion** (pull from databases, APIs, IoT, SaaS), **transformation** (fix errors, deduplicate, normalise, reshape for the consumer) and **serving** (deliver to dashboards, ML datasets, reports).
- Main use cases: centralised data collection and storage, analytics, real-time views of the business and feeding ML training.
- Role split: data scientists predict, data analysts explain the present from prepared datasets, data engineers build the infrastructure both depend on.
- Toolkit: ETL/ELT pipelines; storage (cloud object stores, relational and NoSQL databases, warehouses, lakes, lakehouses); languages (SQL, Python, Scala for Spark, Java for JVM-based systems).
- Observability is part of the job: monitoring pipelines so consumers receive trustworthy data.

**Compare / don't confuse with:** ETL (transform before loading; schema-on-write) vs. ELT (load raw first, transform inside the warehouse; more flexible, enables reprocessing).

**Interview angle**
- *Q: What does a data engineer own that a data scientist doesn't?* — The reliability of the data supply: ingestion, storage design, transformations, scheduling, data quality checks and monitoring. The scientist owns the model; the engineer owns that the inputs are correct, fresh and reproducible.
- *Q: Why land raw data before transforming it?* — So you can reprocess history when business logic changes or a bug is found, without going back to the source.

**Your experience hook:** At MICHU and on the KYC/KYB platform you built the ingestion, CDC and ETL layers across six banks and multiple source databases, and you now run Airflow + Redpanda pipelines that produce analysis-ready and ML datasets.

**Beyond IBM:** IBM's description of Scala ("unlike Python, permits concurrency primitives") is dated; today most pipeline code is SQL (dbt) plus Python, with Spark used through PySpark or Spark SQL. The modern framing is "software engineering applied to data": version control, tests, CI/CD and idempotent, rerunnable jobs.

---

## Agentic AI data engineering
*Source: [IBM — What Is Agentic AI Data Engineering?](https://www.ibm.com/think/topics/agentic-ai-data-engineering)*

**In one line:** Using AI agents (LLM-driven systems that plan steps and call tools) to design, run, monitor and repair data pipelines with limited human oversight.

**Key points**
- Motivation: engineers spend much of their time firefighting. A single upstream schema change or renamed column can break a chain of scripts, stored procedures and scheduled jobs.
- **Pipeline creation:** "declarative pipeline authoring" — a user states what the pipeline should deliver in natural language and the agent works out ingestion, transformations and target. Technical users can steer it through a Python SDK.
- **Job execution:** agents call tools and APIs, read metadata and choose an integration style (batch ETL/ELT, streaming, replication) and runtime (on-prem, cloud, pushdown) per job step.
- **Continuous monitoring:** detect schema drift, anomalies and quality issues; do root-cause analysis; propose and apply fixes (retry, scale up, adjust logic).
- Claimed benefits: handles heterogeneous sources, automates profiling/validation, reduces pipeline sprawl, supports compliance via lineage, gives business users self-service.
- Risks IBM lists: hard to connect to legacy systems, agents doing unexpected things (e.g. reprocessing huge volumes during business hours), and agents contributing to data breaches. Guardrails and governance are the mitigation.

**Interview angle**
- *Q: Would you let an agent auto-fix production pipelines?* — Only inside guardrails: read-only diagnosis by default, changes as reviewed pull requests, automatic actions limited to safe, idempotent operations (retry, rerun a partition), with cost and time-of-day limits and full audit logs.

**Your experience hook:** You have built n8n workflows that orchestrate Claude Code agents, so you can talk concretely about tool-calling agents, what they are good at, and where human review is needed.

**Beyond IBM:** Agent-driven "self-healing" only works safely when the underlying jobs are idempotent and backfillable; an agent that retries a non-idempotent load just duplicates data faster. Data contracts and schema tests give agents a clear signal of what "broken" means.

---

## Data acquisition
*Source: [IBM — What Is Data Acquisition?](https://www.ibm.com/think/topics/data-acquisition)*

**In one line:** Data acquisition is obtaining data from any source and by any means (collecting, converting, exchanging or buying), as the first step before validation, transformation and loading.

**Key points**
- Four methods (from the US Geological Survey): **collect** new data (surveys, sensors, IoT), **convert** legacy data into a usable format, **share/exchange** data (APIs, file transfer, streams, open-data programmes) and **purchase** data from marketplaces.
- Common sources: ERP/CRM applications, social media, open and public data (e.g. census), transactions, surveys, web analytics, IoT.
- Considerations: privacy and consent (GDPR), data quality, compatibility/standardisation (e.g. "North" → "N"; HIPAA code sets) and whether the business value justifies the cost (licences, cleaning, storage, governance).
- An alternative meaning in engineering and science: capturing physical signals with DAQ (data acquisition) devices and converting them to digital values.
- For ML, scarce or restricted data pushes teams towards **synthetic data** that mimics real distributions.

**Compare / don't confuse with:** *Data collection* (gathering raw data directly from sources) is a subset of *data acquisition*, which also includes licensing, partnerships, purchases and legacy conversion.

**Interview angle**
- *Q: What do you check before onboarding a new external data source?* — Legal basis and licence terms, PII content, schema and update frequency, delivery mechanism and SLAs, quality profile on a sample, and the cost of ongoing cleaning versus its value.

**Your experience hook:** Onboarding six banks onto MICHU meant acquiring data from partner systems with different formats and agreements, under privacy constraints for customer data.

---

## Data pipeline
*Source: [IBM — What Is a Data Pipeline?](https://www.ibm.com/think/topics/data-pipeline)*

**In one line:** A data pipeline is a system that takes data from sources, optionally processes it and delivers it to a destination such as a warehouse, lake or application.

**Key points**
- Types: **batch** (scheduled bulk loads, often off-peak; MapReduce lineage), **streaming** (continuous event processing via brokers such as Kafka, often fed by CDC), **data integration** (merging sources into one view) and **cloud-native** pipelines.
- Three stages: **ingestion** (sources are called producers/publishers in streaming), **transformation** (e.g. flattening nested JSON, enrichment) and **storage** (database, warehouse, lakehouse; readers are consumers/subscribers).
- IBM recommends landing raw data first so historical data can be reprocessed when jobs change.
- A pipeline is broader than ETL: it need not follow E→T→L order, can be streaming, and may move data with no transformation at all.
- Use cases: fraud detection, exploratory analysis, dashboards, ML training and inference, observability.
- Design considerations: format variety, throughput, latency, fault tolerance (monitoring, retries) and security (authentication, authorisation, encryption).

**Compare / don't confuse with:** Data pipeline (any movement of data) vs. ETL pipeline (a specific pattern where transformation is required before load).

**Interview angle**
- *Q: Walk me through how you'd design a pipeline.* — Start from the consumer's freshness and correctness requirements; choose batch, streaming or CDC; land raw data immutably; transform in layered, testable steps; make every step idempotent; orchestrate with retries and alerts; add data-quality checks and lineage.
- *Q: How do message brokers avoid losing data?* — Consumers acknowledge (or commit offsets) only after processing, so unacknowledged messages are redelivered. That gives at-least-once delivery, so consumers must handle duplicates.

**Your experience hook:** Your MongoDB → Oracle project combined initial load, full refresh and incremental CDC (inserts, updates, deletes), which covers both batch and streaming pipeline styles.

**Beyond IBM:** IBM says streaming is "not as reliable" as batch; in practice durable logs like Kafka/Redpanda are very reliable. The real design question is delivery semantics (at-most-once, at-least-once, effectively-once) plus idempotent sinks, dead-letter queues for poison messages and a backfill path for reprocessing.

---

## AI data pipeline
*Source: [IBM — What Is an AI Data Pipeline?](https://www.ibm.com/think/topics/ai-data-pipeline)*

**In one line:** An AI data pipeline continuously ingests, prepares and delivers data, including unstructured and real-time data, for training, serving and grounding AI models.

**Key points**
- Traditional batch ETL into a warehouse struggles with AI needs: real-time data, unstructured data for generative AI and RAG, feature stores, semantic context and fast-moving governance. Hard-coded pipelines also break on schema changes and lead to "pipeline sprawl".
- AI pipelines often "shift left" (process closer to the source) to cut latency and keep data fresh.
- Components: ingestion (event streams, APIs; Kafka), preprocessing (streaming transforms; text extraction and **vector embeddings** for unstructured data), **feature engineering**, model training (train/validation/test split, annotation/labelling), scalable storage (object storage, parallel file systems, lake/warehouse/lakehouse), deployment for **online** vs **batch inference**, **context engineering** (semantic models, metadata, business definitions, entity relationships) and governance (access, lineage, bias and drift detection).
- Agents can help build, monitor and fix these pipelines (see the previous entry).
- Use cases: streaming fraud detection, instant personalisation, RAG over fresh documents.

**Compare / don't confuse with:** Online inference (one request at a time, low latency) vs. batch inference (scheduled scoring of many records, cheaper when timeliness doesn't matter).

**Interview angle**
- *Q: What changes when a pipeline feeds a RAG system instead of a dashboard?* — You process unstructured documents (parse, chunk, embed), store vectors with metadata for filtering and access control, and must handle updates and deletes so the index never serves stale or revoked content.

**Your experience hook:** You build customer-facing RAG systems and ML datasets for credit scoring, so you have prepared both features and embeddings for models.

**Beyond IBM:** IBM's "fitting set" is usually called the **validation set**. A key practitioner risk is training/serving skew: features computed differently offline and online. Feature stores and shared transformation code exist to prevent it, and point-in-time correct joins prevent label leakage.

---

## Data pipeline automation
*Source: [IBM — What Is Data Pipeline Automation?](https://www.ibm.com/think/topics/data-pipeline-automation)*

**In one line:** Using software to run, validate, monitor and recover data pipelines with minimal manual work.

**Key points**
- Why: teams spend a large share of engineering time on maintenance and one-off scripts; AI systems need fresh, governed data with known lineage.
- Benefits: fewer human errors, engineering time freed up, timely data, resilience through monitoring and alerting, scaling without proportional headcount, consistent governance.
- Components: **ingestion** triggered by schedule, event or detected change; CDC instead of full refreshes; jobs that support "automatic retries, checkpoint-based restarts and data replay without introducing duplicates".
- **Staging and layering:** raw landing zones with load timestamps and lineage; **bronze/silver/gold** (medallion) layers promoted only after checks pass.
- **Orchestration:** tools like Airflow manage dependencies, order, retries and error handling; pipelines as DAGs allow rerunning only failed parts.
- **Embedded quality checks:** schema, referential integrity, freshness and anomaly rules; bad records are quarantined rather than stopping everything.
- Plus monitoring/observability, active metadata (driving selective reprocessing), and policy-based governance (RBAC/ABAC, masking, retention).
- Implementation advice: set business goals and freshness/latency targets, inventory sources (API limits, schema volatility), choose ETL/ELT and batch/stream, use CI/CD and version control, test under realistic load and failure.

**Interview angle**
- *Q: What makes a pipeline safe to retry automatically?* — Idempotent writes (MERGE/upsert on a business key, or overwrite a whole partition), deterministic inputs (a fixed data interval, not "now"), and checkpoints so a restart resumes rather than double-processes.

**Your experience hook:** After CDC streams stalled silently on the KYC/KYB platform, you added resume tokens and monitoring, which is exactly the checkpoint-restart and observability layer IBM describes.

**Beyond IBM:** Quarantine is often implemented as a dead-letter table or queue with the failing record, the error and the run ID, so it can be fixed and replayed. Kestra (YAML, event-driven) and Dagster (asset-based, with built-in lineage and checks) are common alternatives to Airflow for this kind of automation.

---

## How to automate data pipelines in 10 steps
*Source: [IBM — How to Automate Data Pipelines: A 10-Step Guide](https://www.ibm.com/think/topics/how-to-automate-data-pipelines-in-10-steps)*

**In one line:** A practical sequence for taking a pipeline from purpose to self-maintaining production system.

**Key points**
1. **Define objectives:** use cases, stakeholders and SLAs for latency, freshness and reliability.
2. **Identify and profile sources:** discover, catalogue (type, sensitivity, owner), profile completeness, duplicates and schema consistency.
3. **Design the architecture:** map flows; choose ETL, ELT, reverse ETL, batch, streaming or hybrid; document ownership; prefer declarative, metadata-driven definitions.
4. **Ingest and validate:** CDC for incremental loads; retries and buffering; validate on entry; make ingestion **idempotent** so reruns don't duplicate.
5. **Transform:** SQL, Python, Spark, dbt; choose **full refresh** (simple, correct, expensive) vs **incremental** (cheap, needs safeguards); watch data skew.
6. **Store and organise:** medallion layers, staging for reprocessing, retention policies, columnar formats and partitioning.
7. **Orchestrate:** DAGs in Airflow, Prefect or Dagster; time-, event- or data-availability triggers; parallelism; CI/CD.
8. **Monitor and alert:** freshness, throughput, error rates, null rates and distribution shifts, on both infrastructure and data.
9. **Scale:** distributed compute, workload isolation, query tuning, cost controls.
10. **Operationalise and iterate:** trigger downstream actions, feedback loops, guardrails, documentation.

**Interview angle**
- *Q: Full refresh or incremental?* — Full refresh for small tables or when correctness matters more than cost; incremental for large, append-heavy or CDC sources, with a watermark column, a lookback window for late data and a periodic full rebuild to correct drift.

**Your experience hook:** Your MongoDB → Oracle ETL offered both full-refresh and incremental modes plus an initial-load mode, which is step 5's trade-off in practice.

**Beyond IBM:** Plan the **backfill** path at design time: parameterise every job by its data interval (as Airflow's `data_interval_start/end` does) so any historical window can be rerun without code changes, and protect sources from backfill load with concurrency limits.

---

## Directed acyclic graph (DAG)
*Source: [IBM — What Is a Directed Acyclic Graph (DAG)?](https://www.ibm.com/think/topics/directed-acyclic-graph)*

**In one line:** A DAG is a graph of nodes joined by one-way edges with no cycles, used to express dependencies and order.

**Key points**
- Parts: **nodes** (vertices), **directed edges** (arrows), **paths**; a **collider** is a node with two incoming edges; a **tree** is a DAG where each node has exactly one parent except the root.
- "Acyclic" means you can never follow arrows back to where you started, so there is always a valid execution order.
- **Topological sort** produces a linear order where every node comes after its predecessors.
- **Transitive closure** adds an edge for every indirect reachability (A→B→C implies A→C); **transitive reduction** removes redundant edges while keeping the same reachability.
- Uses: data pipelines (Airflow defines DAGs in Python), neural network structure, causal inference (spotting confounders), robot task planning, compiler optimisation (common subexpressions), DAG-based blockchains.

**Interview angle**
- *Q: Why must a workflow be acyclic?* — A cycle means a task depends on itself, so no valid start order exists. Acyclicity guarantees a topological order, lets the scheduler run independent branches in parallel and lets you rerun just a failed node and its downstream nodes.
- *Q: How would you find execution order?* — Kahn's algorithm: repeatedly run nodes with no unmet dependencies; if nodes remain but none are ready, there is a cycle.

**Beyond IBM:** Spark and dbt also build DAGs internally (Spark's stage plan; dbt's `ref()` graph), and Dagster models the graph as data assets rather than tasks. Transitive reduction is what you want when drawing lineage: fewer edges, same meaning.

---

## Data cleaning
*Source: [IBM — What Is Data Cleaning?](https://www.ibm.com/think/topics/data-cleaning)*

**In one line:** Data cleaning (cleansing, scrubbing) finds and fixes errors and inconsistencies in raw data so it is accurate, complete, consistent and usable.

**Key points**
- Targets: duplicates, missing values, inconsistent formats, syntax errors, irrelevant data and structural errors.
- Cleaning raises quality; transformation then reshapes clean data for use. Dirty inputs make BI and ML outputs unreliable or biased.
- Starts with **profiling** (assessment), then techniques: **standardisation** (e.g. one date format), **outlier handling** (decide whether error or real, then keep, adjust or remove), **deduplication**, **missing values** (impute, drop or flag) and a final **validation** pass.
- Benefits: better decisions, productivity, lower cost of errors, GDPR accuracy and less retained sensitive data, better model performance, consistent integration.
- AI helps: suggesting rules from patterns, NLP for addresses, choosing the "surviving" record during dedup (e.g. most recent email), learning rules from past corrections.

**Interview angle**
- *Q: How do you deduplicate records with no shared ID?* — Normalise fields, then entity resolution: block on cheap keys to limit comparisons, score candidate pairs with fuzzy/probabilistic matching, cluster matches and apply survivorship rules to build the golden record.

**Your experience hook:** You used Splink for entity resolution, where blocking cut 14.7B candidate pairs to 77M, and fixed ID and phone-number collisions with UUID surrogate keys.

**Beyond IBM:** In pipelines, clean in code, not by hand: keep the raw layer untouched and apply cleaning as versioned, tested transformations so results are reproducible and auditable.

---

## Data reconciliation
*Source: [IBM — What Is Data Reconciliation?](https://www.ibm.com/think/topics/data-reconciliation)*

**In one line:** Data reconciliation compares data across systems to confirm they agree, and resolves the differences.

**Key points**
- Typically runs after data is moved, migrated or transformed, alongside or after ETL.
- Process: **extract** from each side → **standardise** (formats, names, dedup) → **compare** → **identify and classify** discrepancies by severity → **resolve** (automatic rules or data stewards) → **validate and log** an audit trail.
- Types: manual vs automated; **transaction-level** (match each record, e.g. bank statement vs ledger), **balance-level** (compare totals; fast but can hide offsetting errors), **system-to-system** (CRM vs ERP, migrations).
- Use cases: patient records in healthcare, regulatory reporting and trade matching in finance, SWIFT/ACH vs internal logs for fraud, inventory and orders in supply chain.
- Efficiency tips: reconcile access/config data separately, pick a cadence that matches change rate, reconcile only changed records, compare only needed attributes.

**Compare / don't confuse with:** *Validation* checks single records against rules; *reconciliation* compares datasets across systems after the fact (corrective); *synchronisation* keeps systems aligned continuously (preventive).

**Interview angle**
- *Q: How do you prove a migration or CDC pipeline didn't lose data?* — Compare row counts and sums per partition or day, then hash key columns per row (or per bucket) on both sides to find exactly which records differ, and log results for audit.

**Your experience hook:** Consolidating 2M+ KYC/KYB records from multiple source databases via CDC, and dealing with streams that stalled silently, is a natural reconciliation story: counts and checksums are what reveal silent gaps.

**Beyond IBM:** Make reconciliation a scheduled pipeline task with alerting, not a one-off script. Balance-level checks catch missing batches cheaply; hash-based row diffs pinpoint the records.

---

## Data validation
*Source: [IBM — What Is Data Validation?](https://www.ibm.com/think/topics/data-validation)*

**In one line:** Data validation checks that data meets defined rules (types, formats, ranges, business logic) before it is stored or used.

**Key points**
- Valid data is within allowed ranges, in the right format, accurate and compliant with the organisation's own criteria.
- Matters more now because of scale, migrations, AI training (including data poisoning risk) and regulation such as the EU AI Act for high-risk systems.
- Common checks: **code** (value in an allowed list, e.g. country codes), **consistency** (engagement date before wedding date), **data type**, **format** (email, phone, date), **range** (min/max), **uniqueness**. Also length, presence (not null) and schema checks.
- Validation and cleansing are both parts of **data quality management**, with profiling, monitoring and metadata management.
- Tools: spreadsheet rules, Python/SQL scripts, integration platforms (ETL is known for strict validation) and observability tools.

**Compare / don't confuse with:** Validation (detects whether data passes rules) vs. cleansing (changes data to fix it).

**Interview angle**
- *Q: Where in a pipeline do you validate, and what happens to failures?* — At ingestion (schema and contract checks), after each transformation (uniqueness, not-null, referential integrity) and before publishing (freshness, volume anomalies). Critical failures block publication; row-level failures go to a quarantine table with the reason.

**Beyond IBM:** Typical tools are dbt tests, Great Expectations, Soda or Pandera. Distinguish "hard" checks that stop the pipeline from "soft" checks that only warn, and test schemas at producer boundaries with data contracts.

---

## Data enrichment
*Source: [IBM — What Is Data Enrichment?](https://www.ibm.com/think/topics/data-enrichment)*

**In one line:** Data enrichment adds new attributes from internal or external sources to an existing dataset to give it more context and value.

**Key points**
- Example: customer names and phones enriched with emails; addresses enriched with coordinates (geocoding).
- Benefits: fills gaps, raises trust, improves model performance, supports decisions and, through metadata enrichment, compliance tracking.
- Types: behavioural, contact, demographic, firmographic (company), geographic, psychographic (interests, attitudes), technographic (tech used).
- Steps: clean and deduplicate the target first → decide what to add → pick sources → join it in with integration tooling.
- Sources: first-party internal data (often siloed, needs integration), public data (census), third-party vendors (vet them). Enriched data stays subject to GDPR/HIPAA.
- Use cases: marketing segmentation, urban planning, healthcare (wearables), cybersecurity (adding location and device to security events).

**Compare / don't confuse with:** *Enhancement* improves the data you already have (clean, update, fill some gaps); *enrichment* appends new fields from other sources.

**Interview angle**
- *Q: What can go wrong when enriching with a third-party feed?* — Join keys don't match (so you must resolve entities first), the feed changes or arrives late, licensing restricts use, and enrichment can add PII you must now govern. Enrich with point-in-time data so historical records aren't rewritten with today's values.

**Beyond IBM:** In streaming, enrichment is usually a stream-table join (event joined to a compacted reference topic or cache such as Redis) and needs a policy for missing or late reference data.

---

## Data wrangling
*Source: [IBM — What Is Data Wrangling?](https://www.ibm.com/think/topics/data-wrangling)*

**In one line:** Data wrangling (munging, data preparation) is the hands-on process of cleaning, restructuring and enriching raw data into an analysis-ready dataset.

**Key points**
- Five steps: **discover** (profile sources and quality, write a profiling report) → **structure** (aggregate, pivot, join, convert types) → **clean** (missing values, duplicates, noise; avoid over-cleaning) → **enrich** (add relevant external data, e.g. weather for shipment delays) → **validate** (type, range/format, consistency, uniqueness, cross-field, statistical checks).
- Then publish: load to a warehouse, build visuals or export for ML.
- Often cited as taking roughly 45–80% of an analyst's time.
- Tools: Python, R, SQL; spreadsheets for small data; visual prep tools; Spark/Hadoop for big data; AI for outlier detection and natural-language interfaces.

**Compare / don't confuse with:** Wrangling is usually exploratory and analyst-driven for a specific analysis; a data pipeline is the automated, repeatable version for production use.

**Interview angle**
- *Q: How do you turn an analyst's wrangling notebook into production?* — Extract the logic into versioned SQL/Python transformations, parameterise inputs, add tests for the assumptions the analyst checked by eye, and schedule it in the orchestrator.

**Your experience hook:** Preparing credit-scoring inputs with MongoDB aggregation pipelines before factor analysis and PCA is a wrangling-to-production example.

---

## Data virtualization
*Source: [IBM — What is Data Virtualization?](https://www.ibm.com/think/topics/data-virtualization)*

**In one line:** Data virtualization puts a query layer over many sources so users can access combined data in place, without copying it into one store.

**Key points**
- A virtual layer sits between sources and consumers and fetches data on demand, so there is no replication or movement.
- Benefits: faster and cheaper integration, a single point of access across real-time and historical data (e.g. sensor streams plus ERP/CRM history), agility.
- Design issues: **latency** (remote reads are slower than local; tune networks and add refresh strategies), **source strain** (constant queries can overload operational systems, so fetch only critical fields, batch or replicate where needed), the **abstraction** benefit for developers, and storage trade-offs in hybrid setups that mix ingested and virtualised data.
- Use cases: supply chain visibility, customer 360 analytics, fraud detection.

**Compare / don't confuse with:** Virtualization (query data where it lives) vs. ETL/ELT (copy data into a central store). Virtualization is quick to set up and always current, but query speed and source load depend on the slowest source.

**Interview angle**
- *Q: When would you choose federation over building a pipeline?* — For exploration, low-volume or infrequent queries, or data that can't legally be moved. For heavy, repeated analytics or SLAs, materialise the data so you don't hammer production systems.

**Beyond IBM:** Open-source equivalents are Trino/Presto federated queries and PostgreSQL foreign data wrappers. Pushdown (sending filters to the source) is what makes or breaks performance.

---

## Data automation
*Source: [IBM — What Is Data Automation?](https://www.ibm.com/think/topics/data-automation)*

**In one line:** Data automation removes manual steps from data handling (integration, extraction, transformation, loading, validation and analysis) using tools and code.

**Key points**
- Motivation: huge data volumes, much of it never analysed; hand-written scripts per source were fragile and costly.
- Works by building a pipeline that collects, processes and delivers data automatically, using connectors, RPA for repetitive UI tasks and ML for complex extraction and adaptive transformations.
- Lifecycle steps automated: **integration** (umbrella), **extraction** (including documents, emails, even handwriting), **transformation** (clean, reformat, aggregate, enrich), **loading** and **analysis** (encoding, imputation, splitting, visualisation).
- Loading pattern: an **initial full load**, then periodic **incremental** loads, with occasional **full refreshes**; triggered by time or by events (new file arrives, record changes).
- Benefits: efficiency, fewer human errors, cost savings, faster insights and decisions, security (automatic encryption, auditing, access control), scalability.

**Compare / don't confuse with:** Data automation (making individual data tasks run without people) vs. data orchestration (coordinating many automated tasks, their order and dependencies).

**Interview angle**
- *Q: Describe a load strategy for a new source table.* — Initial bulk load, then incremental loads by CDC or a watermark column, with idempotent upserts; schedule a periodic full refresh or reconciliation to catch missed changes such as hard deletes.

**Your experience hook:** Your MongoDB → Oracle ETL implemented exactly this trio: initial load, incremental change streams (including deletes) and full refresh.

---

## Data orchestration
*Source: [IBM — What Is Data Orchestration?](https://www.ibm.com/think/topics/data-orchestration)*

**In one line:** Data orchestration coordinates data tasks across systems: what runs, in what order, when, and what happens when something fails.

**Key points**
- Why: data spans many systems (IDC: on average 35 source systems feeding 18 analytical repositories), creating silos, staleness and errors.
- Three steps: **organise** (collect sources), **transform** (unify, clean, validate), **activate** (deliver to dashboards and apps).
- Functions: define dependencies and sequence (usually as DAGs, or code-first Python flows for dynamic workflows), automate workflows, monitor and alert, with **automatic retries** before notifying people.
- Trigger types: **time-based**, **dependency-based**, **event-driven** (e.g. API call, file arrival).
- Benefits: breaks silos, better quality and freshness, scalability, faster insights, AI readiness, lineage for compliance, less manual work.
- Tools: Apache Airflow (most known, batch-oriented, DAGs in Python), AWS Step Functions (serverless, visual), Azure Data Factory (managed ETL/ELT), Dagster (observability, lineage, testing), Prefect (code-first).

**Compare / don't confuse with:** Orchestration (the conductor) vs. integration (one of the jobs it conducts, such as ETL). Data orchestration is a type of *workflow orchestration*, which in turn is narrower than *process orchestration* of whole business processes.

**Interview angle**
- *Q: What should an orchestrator do, and what shouldn't it do?* — It should schedule, manage dependencies, pass parameters such as the data interval, retry, alert and record run history. It shouldn't do the heavy data processing itself; push that to the warehouse, Spark or the database.
- *Q: Airflow vs Dagster vs Kestra?* — Airflow: mature, task-centric Python DAGs, huge operator ecosystem. Dagster: asset-centric (you declare the tables you produce), strong lineage, testing and data checks. Kestra: declarative YAML flows, event-driven triggers, language-agnostic tasks.

**Your experience hook:** You run Airflow in production for the Redpanda ETL and have used Kestra in the Zoomcamp, and n8n for agent workflows, so you can compare them from use.

**Beyond IBM:** IBM's claim that Prefect "does not rely on DAGs" is loose: Prefect builds the dependency graph at runtime from Python code, while Airflow also supports dynamic task mapping. Key Airflow concepts to know: `catchup` and backfills over data intervals, `retries` with `retry_delay`, sensors and deferrable operators, pools for limiting concurrency, and keeping tasks idempotent so reruns are safe.

---

## Self-test

1. What are the three phases of the data integration pipeline IBM describes, and why should raw data be landed before transforming it?
<details><summary>Answer</summary>Ingestion, transformation and serving. Landing raw data immutably lets you reprocess history when logic changes or a bug is found, without re-extracting from the source.</details>

2. How does a data pipeline differ from an ETL pipeline?
<details><summary>Answer</summary>ETL is one specific pattern with mandatory transformation before load, usually batch. A data pipeline is any system moving data: it can be ELT, streaming, or pure movement with no transformation.</details>

3. What does it mean for an ingestion job to be idempotent, and how do you achieve it?
<details><summary>Answer</summary>Running it more than once for the same input gives the same result, with no duplicates. Use upserts/MERGE on a business key or overwrite a whole partition, and parameterise runs by a fixed data interval rather than "now".</details>

4. Give the trade-off between full refresh and incremental processing.
<details><summary>Answer</summary>Full refresh is simple and self-correcting but costly and slow on large data. Incremental is efficient but needs a watermark or CDC, handling of late data and deletes, and periodic reconciliation or rebuilds to catch drift.</details>

5. Why must orchestration workflows be acyclic, and what algorithm orders them?
<details><summary>Answer</summary>A cycle would make a task depend on itself, so no valid start order exists. A topological sort (e.g. Kahn's algorithm) orders tasks so every task runs after its dependencies and independent branches can run in parallel.</details>

6. Distinguish data validation, data reconciliation and data synchronisation.
<details><summary>Answer</summary>Validation checks individual records against rules (format, range, uniqueness). Reconciliation compares datasets across systems after movement to find and fix mismatches. Synchronisation continuously propagates changes to keep systems aligned.</details>

7. Name four common validation checks with an example each.
<details><summary>Answer</summary>Code check (country code in ISO list), range check (tyre pressure 30–35 psi), format check (valid email pattern), uniqueness check (no duplicate customer IDs). Also type, consistency, presence, length and schema checks.</details>

8. What is the difference between data enrichment and data enhancement?
<details><summary>Answer</summary>Enhancement improves existing data (cleaning, updating, filling some gaps). Enrichment appends new attributes from other internal or external sources, such as geocodes or firmographics.</details>

9. When would you use data virtualization instead of an ELT pipeline?
<details><summary>Answer</summary>For ad hoc or low-volume access, quick prototypes, or data that can't be copied. For heavy, repeated or SLA-bound workloads, materialise the data to avoid latency and load on source systems.</details>

10. Name the three trigger types for orchestrated tasks and one thing an orchestrator should not do.
<details><summary>Answer</summary>Time-based, dependency-based and event-driven triggers. An orchestrator should not do the heavy data processing itself; it should delegate to the warehouse, Spark or database and just coordinate, retry and alert.</details>
