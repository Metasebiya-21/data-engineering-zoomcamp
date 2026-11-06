# 1. Introduction

> This section sets the frame for everything else: what "data management" covers end to end (storage, integration, governance, security, observability, master data), how the cloud-based "modern data stack" replaced on-prem stacks, what makes data usable for AI, how AI is now used to manage data, and how DataOps brings DevOps habits (automation, CI/CD, monitoring) to pipelines. Interviewers use these topics to check whether you can see the whole platform rather than one tool, and whether you can talk about reliability, ownership and trade-offs like a lead.

## Data management
*Source: [IBM — What Is Data Management?](https://www.ibm.com/think/topics/data-management)*

**In one line:** Data management is the whole discipline of collecting, storing, integrating, securing and governing data so that it is reliable and usable for business, analytics and AI.

**Key points**
- A full strategy covers: ingestion from many sources and clouds; availability and disaster recovery; fit-for-purpose databases; sharing data and metadata; security, governance and compliance; and lifecycle management (lineage, observability, MDM) from creation to deletion.
- Storage follows the workload: relational/transactional databases for operations, NoSQL (key-value, document, wide-column, graph, in-memory) for other shapes, warehouses for schema-defined analytics, lakes for cheap raw storage, lakehouses for open formats on object storage with warehouse-like querying.
- Integration happens through ETL, ELT (growing with cloud platforms) and replication; CDC reads the source's log to push changes to targets so decisions use current data.
- Batch is still more efficient for large volumes and fine for dashboards; streaming (for example Kafka topics) gives seconds-level freshness.
- Governance sets the rules (access to PII, taxonomies, catalogs, lineage, SLAs); data management implements them. Governance is described as a subset of data management.
- Data fabric = automated, metadata-driven integration across silos; data mesh = decentralised ownership by business domain.
- Other building blocks: security (encryption, masking), observability (diagnosing issues, not just alerting), MDM (one trusted record per customer or product), and semantic/shared metadata layers.

**Compare / don't confuse with:** Data governance (policies, roles, who may access what) vs. data management (the systems and processes that carry those policies out). Data fabric (an integration architecture) vs. data mesh (an organisational ownership model).

**Interview angle**
- *Q: What does a data management strategy need to cover beyond "build pipelines"?* — Storage fit for each workload, integration (batch, streaming, CDC), quality and observability, governance and access control, security, lineage, master data, and lifecycle/retention, plus recovery from failure.
- *Q: When would you choose batch over streaming?* — When consumers do not need sub-minute freshness (most reporting), because batch is cheaper, simpler to re-run and easier to reason about; stream only where latency changes the outcome.

**Your experience hook:** Consolidating 2M+ records from several source databases with CDC, and later adding resume tokens and monitoring after streams stalled silently, touches integration, observability and master data in one project.

**Beyond IBM:** The lakehouse in practice means an open table format (Apache Iceberg, Delta Lake or Hudi) on object storage, which adds ACID commits, schema evolution and time travel on top of Parquet files. CDC is not "set and forget": you must handle deletes, ordering, schema changes and restarts from a saved offset.

---

## Modern data stack
*Source: [IBM — What Is the Modern Data Stack?](https://www.ibm.com/think/topics/modern-data-stack)*

**In one line:** The modern data stack (MDS) is a modular set of cloud-native tools that together ingest, store, transform, analyse, monitor and govern data.

**Key points**
- Five core layers: storage (warehouse, lake, lakehouse), ingestion (batch or streaming), transformation (cleaning, normalising, aggregating, merging), BI and analytics, and data observability.
- Extra layers often added: data discovery, governance, cataloguing, and ML/AI.
- It grew up roughly 2012–2016 with cloud warehouses (Snowflake, BigQuery, Redshift), the shift from ETL to ELT, managed ingestion (Fivetran, Airflow), SQL transformation tools (dbt, Dataform) and BI tools (Tableau, Power BI, Looker).
- Reverse ETL pushes modelled data from the warehouse back into operational tools such as CRMs.
- Legacy vs. modern: physical servers vs. cloud; manual vs. elastic scaling; custom scripts vs. automated connectors; monolith vs. modular; batch reporting vs. near-real-time; capex vs. pay-as-you-go.
- Batch is the most common ingestion mode because it is cheaper; streaming costs more compute but is needed for things like fraud detection.

**Compare / don't confuse with:** ETL (transform before loading, typical of legacy stacks) vs. ELT (load raw first, transform inside the warehouse, typical of the MDS). Reverse ETL is the opposite direction: warehouse to operational apps.

**Interview angle**
- *Q: Why did ELT replace ETL in the modern stack?* — Cloud warehouses made storage cheap and compute elastic, so it became easier to land raw data first and transform it with SQL in place, keeping the raw copy for reprocessing.
- *Q: What are the risks of a modular stack?* — Tool sprawl, many vendor bills, integration seams between tools, and unclear ownership when something breaks between layers.

**Your experience hook:** The Zoomcamp material (BigQuery for storage, dbt for transformation, Kestra/Airflow for orchestration, Terraform for infrastructure) is a hands-on MDS, and your current Airflow + Redpanda pipelines add the streaming ingestion layer.

**Beyond IBM:** "Pay-as-you-go" can cost more than expected: warehouse spend needs partitioning, clustering and query discipline. IBM's claim of "no vendor lock-in" is optimistic; open table formats and portable SQL (dbt) are what actually reduce lock-in.

---

## AI-ready data
*Source: [IBM — What Is AI-Ready Data?](https://www.ibm.com/think/topics/ai-ready-data)*

**In one line:** AI-ready data is data that is trustworthy, accessible, governed and secured well enough to be used safely for training or feeding AI systems.

**Key points**
- Four traits: unified and accessible, governed, secure, and supported (by skills and infrastructure).
- Barriers: data sprawl and silos, poor quality, skills gaps and overloaded teams, and security/governance risk from sensitive data scattered everywhere.
- Most enterprise data is unstructured (PDFs, emails, chats, images), and very little of it is in a form AI can consume directly; this is a major untapped source.
- Unified access comes from data integration and data fabric (a virtual view across sources without physically moving everything).
- Governance for AI: access controls, documented lineage, bias checks, validation and observability, and metadata that describes datasets. Regulations such as the EU AI Act require documentation of provenance and fitness for purpose.
- Security for AI follows three steps: discover and classify sensitive data, protect it (encryption, backups, DR), and monitor activity. Gen AI adds new risks such as data leakage and prompt injection.
- The introduction article adds a sharp point: AI-ready is not the same as "clean". Analytics data is often stripped of outliers, but a model needs representative data, including errors and edge cases it will meet in production.

**Compare / don't confuse with:** High-quality data for BI (filtered, conformed to human expectations) vs. AI-ready data (representative of the real use case, including outliers).

**Interview angle**
- *Q: How would you make a document corpus ready for a RAG system?* — Find and classify sources, remove or mask sensitive data, extract text with structure, chunk and embed it with metadata (source, date, permissions), and keep it fresh and access-controlled so answers respect who is asking.
- *Q: Why can "cleaned" data hurt a model?* — If cleaning removes rare but real cases, the model never learns them and fails on them in production.

**Your experience hook:** Building customer-facing RAG systems and ML datasets from Airflow + Redpanda pipelines is exactly the work of turning raw, scattered data into AI-ready inputs.

**Beyond IBM:** In practice AI readiness also needs a held-out evaluation set, dataset versioning so a model can be traced to the data it saw, and checks for train/test leakage.

---

## AI data management
*Source: [IBM — What is AI Data Management?](https://www.ibm.com/think/topics/ai-data-management)*

**In one line:** AI data management means applying AI and ML to the data management lifecycle itself, for example to discover, classify, clean, integrate and protect data.

**Key points**
- Two kinds of AI help: rule-based systems (for example, auto-tagging by predefined patterns) and generative AI (natural-language queries, summaries, extraction).
- Discovery: automated scanning and indexing of storage; classification by rules or ML (a pattern match that flags US social security numbers); LLMs pulling structured fields out of documents. Undiscovered "shadow data" is a real breach risk.
- Quality: automated validation and correction, synthetic data to fill gaps, ML-based duplicate detection in MDM, auto-generated lineage.
- Accessibility: tools that detect relationships between datasets, auto-written catalogue descriptions, natural-language interfaces to catalogues and databases, ML-driven query optimisation.
- Security: automatic PII detection and data loss prevention, anomaly detection (UEBA, EDR), help drafting RBAC policies, fraud detection.
- Benefits: less manual work, better compliance (GDPR, PCI DSS), more people able to use data, and better training data for the organisation's own models.

**Compare / don't confuse with:** AI data management (AI used *to manage* data) vs. AI-ready data / data management *for* AI (preparing data so AI can use it).

**Interview angle**
- *Q: Where would you trust ML in a data platform, and where not?* — Good for suggestions at scale (PII detection, duplicate candidates, anomaly alerts, catalogue descriptions); risky for silent automatic changes to critical records, so keep a human review or clear thresholds and an audit trail.

**Your experience hook:** Your Splink entity-resolution project is ML-based duplicate detection: probabilistic matching with blocking that cut 14.7B candidate pairs to 77M, the same idea IBM describes for MDM tools.

**Beyond IBM:** Synthetic data does not automatically remove privacy risk; generators can memorise and leak real records, so it still needs privacy evaluation before sharing.

---

## DataOps
*Source: [IBM — What Is Dataops?](https://www.ibm.com/think/topics/dataops)*

**In one line:** DataOps is a set of DevOps-style practices (automation, testing, monitoring, collaboration) applied to data pipelines so data is delivered quickly, reliably and with known quality.

**Key points**
- Seven principles: cross-functional collaboration, automation, continuous improvement through metrics, an end-to-end lifecycle view, observability and validation, governance and access control, and data treated as a product for self-service.
- Lifecycle: ingest → orchestrate (transform, align schemas) → validate (automated tests, statistical anomaly checks) → deploy (deliver data products) → monitor (performance, uptime, quality).
- Platform capabilities: scalable ingestion, transformation, metadata and lineage, governance, real-time observability, workflow orchestration.
- Rollout: assess the landscape, form a cross-functional team, define workflows/KPIs/access, add automation and observability, iterate on metrics.
- Challenges: culture change, uneven skills, tool overlap creating new silos, and pilots that do not scale.
- The motivating failure: a single upstream schema change can break a dashboard if teams and checks are not aligned.

**Compare / don't confuse with:** DevOps speeds up software delivery (build-test-deploy of code); DataOps speeds up data delivery (pipelines, tests on data, data products). Both use agile and CI/CD, but DataOps also has to test the data, not just the code.

**Interview angle**
- *Q: What does DataOps look like day to day on your team?* — Pipelines in version control with code review, CI that runs unit and data tests, orchestrated and idempotent jobs, alerts on freshness, volume and schema, and clear owners for each data product.
- *Q: How do you stop an upstream schema change from breaking consumers?* — Contract or schema checks at ingestion, alerting on drift, and versioned, backward-compatible changes agreed with the producer.

**Your experience hook:** After CDC streams stalled silently you added resume tokens plus Prometheus/Grafana monitoring, which is the DataOps "monitor" stage learned the hard way.

**Beyond IBM:** IBM skips the two habits that matter most in practice: idempotent tasks (safe to re-run) and easy backfills. Tools like dbt tests or Great Expectations cover the "validate" stage; data contracts formalise the producer–consumer agreement.

---

## DataOps framework
*Source: [IBM — What Is a DataOps Framework?](https://www.ibm.com/think/topics/dataops-framework)*

**In one line:** A DataOps framework is the concrete set of practices, roles, processes and tools an organisation uses to put DataOps principles into practice.

**Key points**
- Hierarchy: DataOps is the discipline; the framework is the method for applying it; architecture and tools make it real.
- Core components: workflow orchestration and automation (including dependency management so steps run in order), data governance (with data quality management inside it), CI/CD, monitoring and observability, and cross-team collaboration.
- CI/CD for data relies on version control (easy rollback) and automated testing at unit, integration and end-to-end level.
- Observability collects logs, events and metrics (processing time, resource use, error rates) and can also reveal access patterns relevant to privacy compliance.
- Benefits: better quality, faster time to insight, efficiency, agility.
- Example steps: understand business needs → set objectives and roadmap → form teams → choose practices, platforms and tools → build governance (access, formats, lineage, MDM, naming conventions, enforced by automation) → monitor and iterate (schema, lineage, volume).

**Compare / don't confuse with:** DataOps (the idea) vs. DataOps framework (the operating method) vs. DataOps architecture (the systems that run it).

**Interview angle**
- *Q: You join a team with fragile pipelines. What do you do first?* — Assess what exists and where it breaks, agree on priorities with the business, then add version control, tests and monitoring to the critical pipelines before adding new features.

**Your experience hook:** Leading five engineers on the KYC/KYB platform involved the same pieces: shared conventions, CI/CD, monitoring and access control applied across a team.

---

## DataOps architecture
*Source: [IBM — What Is a DataOps Architecture?](https://www.ibm.com/think/insights/dataops-architecture)*

**In one line:** A DataOps architecture is the set of systems, tools and processes that make DataOps repeatable, so pipelines are built and run with consistent speed, quality and governance.

**Key points**
- Without it, DataOps stays as "best intentions": scripts that depend on a few experts and manual checks.
- Benefits: faster delivery, better reliability (tests, monitoring), more trust (metadata, lineage, quality controls), and scalability through modular design.
- Legacy vs. DataOps architecture: designed for stable flows vs. frequent change and schema evolution; manual configuration vs. automation; tools split by team vs. shared visibility; problems found in reports vs. observability in the pipeline.
- Components: data sources (databases, APIs, IoT, feeds); ingestion (batch ETL to streaming, with validation, schema checks and metadata capture on arrival); storage (warehouse, lake, NoSQL, object storage, with access control embedded); processing and transformation (orchestrated, monitored, tested); modelling and computation (analytics and ML, with version control).
- Implementation: assess current state → define target state → pick technology → embed governance → automate integration with reusable pipeline patterns → build shared ownership → monitor with logs, metrics and traces.
- Architecture enables collaboration but does not create it; ownership of data products must be explicit.

**Compare / don't confuse with:** Data architecture (how data is collected, transformed, stored and delivered) vs. DataOps architecture (a data architecture designed for continuous change, automation and observability).

**Interview angle**
- *Q: Walk me through the architecture of a pipeline you own.* — Name the sources, the ingestion pattern and why (batch, CDC or stream), where it lands, how it is transformed and orchestrated, how it is tested and monitored, and who consumes it and with what freshness SLA.

**Your experience hook:** Your current Airflow + Redpanda ETL into analysis-ready and ML datasets maps directly onto these layers: sources, streaming ingestion, storage, orchestrated transformation and ML consumption.

**Beyond IBM:** Reliable ingestion also needs a schema registry or contract checks, dead-letter handling for bad records and idempotent writes so retries do not duplicate data.

---

## Self-test

1. What is the relationship between data governance and data management?
<details><summary>Answer</summary>Governance defines the policies, roles and standards (for example who may access PII); data management builds and runs the systems that carry them out. IBM treats governance as a subset of data management.</details>

2. Name the five core layers of the modern data stack.
<details><summary>Answer</summary>Storage, ingestion, transformation, BI/analytics, and data observability. Discovery, governance, cataloguing and ML/AI are often added on top.</details>

3. What is the difference between ETL, ELT and reverse ETL?
<details><summary>Answer</summary>ETL transforms data before loading it into the target; ELT loads raw data first and transforms it inside the warehouse/lakehouse; reverse ETL sends modelled warehouse data back into operational tools such as a CRM.</details>

4. How does CDC differ from scheduled batch extraction?
<details><summary>Answer</summary>CDC reads the source's change log and propagates inserts, updates and deletes as they happen, keeping targets current with low load on the source. Batch extraction queries the source on a schedule, so data is only as fresh as the last run and deletes are harder to detect.</details>

5. Why is AI-ready data not the same as "clean" analytics data?
<details><summary>Answer</summary>Analytics data is often filtered to remove outliers; a model needs representative data, including the errors and edge cases it will see in production, or it will fail on them.</details>

6. What are the four traits IBM uses to describe AI-ready data?
<details><summary>Answer</summary>Unified and accessible, governed, secure, and supported (by skills and suitable infrastructure).</details>

7. Give two examples of AI used to manage data, and one risk.
<details><summary>Answer</summary>Examples: automatic PII classification, ML-based duplicate detection in MDM, natural-language catalogue search, anomaly detection on access patterns. Risk: silent automatic changes to critical records without review or audit trail.</details>

8. List the five stages of the DataOps lifecycle.
<details><summary>Answer</summary>Ingest, orchestrate, validate, deploy, monitor.</details>

9. How does DataOps differ from DevOps?
<details><summary>Answer</summary>DevOps speeds up and stabilises software delivery (build, test, deploy code). DataOps applies the same automation and CI/CD ideas to data pipelines, and must also test and monitor the data itself (freshness, volume, schema, quality).</details>

10. What is the difference between a DataOps framework and a DataOps architecture?
<details><summary>Answer</summary>The framework is the operating method: practices, roles, processes and steps. The architecture is the set of systems and tools (sources, ingestion, storage, processing, modelling, plus orchestration and observability) that runs it and makes it repeatable.</details>
