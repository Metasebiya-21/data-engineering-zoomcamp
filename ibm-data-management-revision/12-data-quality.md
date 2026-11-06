# 12. Data quality

> This section covers what "good data" means and how to keep it that way: the quality dimensions, the management practices around them, related ideas (accuracy, integrity, reliability, observability) and the named failure modes (bad, stale, dark and dirty data), ending with what changes when the consumer is an AI model. Expect it in almost every data engineering interview, usually as "how do you know your pipeline's output is correct?" or "tell me about a data incident". Strong answers name the dimension, the concrete check (dbt test, freshness check, row-count reconciliation, uniqueness constraint) and the process around it (owner, alert, post-mortem).

## Data quality
*Source: [IBM — What Is Data Quality?](https://www.ibm.com/think/topics/data-quality)*

**In one line:** Data quality is how well a dataset meets agreed criteria such as accuracy, completeness, validity, consistency, uniqueness, timeliness and fitness for its intended use.

**Key points**
- Poor quality is expensive (Gartner estimates an average of USD 12.9M a year per organisation), and "garbage in, garbage out" applies directly to ML models.
- When data meets its standard, consumers can trust it. When it doesn't, quality tooling and root-cause analysis help find and fix the underlying issue.
- Dimensions as IBM lists them here: **completeness** (share of usable, non-missing data), **uniqueness** (no duplicates, e.g. one ID per customer), **validity** (matches formats, types, ranges, patterns), **timeliness** (ready when expected), **accuracy** (matches the agreed source of truth), **consistency** (agrees across datasets and with logical relationships, e.g. department headcount can't exceed company headcount) and **fitness for purpose**.
- For accuracy you name one primary source and use others to confirm it, for example checking that they trend the same way.
- Benefits: better decisions and KPIs, better processes (e.g. supply chain), better customer insight.

**Compare / don't confuse with:** *Data quality vs. data integrity vs. data profiling* — quality is the broad set of criteria; integrity is a subset (accuracy, consistency, completeness) seen through a security lens (preventing corruption or tampering); profiling is the process of examining data to measure quality.

**Interview angle**
- *Q: How do you define "good data" for a table?* — Agree with its consumers which dimensions matter for that use, turn each into a measurable rule with a threshold (e.g. `customer_id` unique and not null, loaded by 06:00, revenue reconciles to the source within 0.1%), and automate those checks in the pipeline.

**Your experience hook:** Replacing colliding IDs and phone numbers with UUID surrogate keys was a fix for uniqueness and accuracy failures that only showed up once several sources were merged.

---

## Data quality management (DQM)
*Source: [IBM — What Is Data Quality Management?](https://www.ibm.com/think/topics/data-quality-management)*

**In one line:** DQM is the set of practices an organisation uses to improve and maintain the quality of its data over time.

**Key points**
- Without it you get wrong marketing targeting, research data that can't be compared and regulatory breaches (GDPR, SOX). Trusted data roughly doubles AI ROI, according to IBM research.
- Six target dimensions: accuracy, completeness, consistency, timeliness, uniqueness, validity. Extra ones include accessibility, relevance, concise representation and appropriate volume.
- **Data profiling:** inspect structure and content to find anomalies and set a baseline.
- **Data cleansing:** standardise formats, handle outliers, deduplicate, deal with missing values.
- **Data validation:** check data against rules (ranges, referential integrity) before use.
- **Monitoring:** quality decays through schema changes, staleness and duplicates, so checks must run continuously. **Metadata management:** record rules, definitions and lineage.
- Tools offer predefined and custom checks, catalogs with quality scores, incident dashboards, real-time alerts, root-cause analysis and lineage. ML is used for anomaly detection.

**Compare / don't confuse with:** *DQM vs. MDM vs. data governance* — DQM improves quality across all data; MDM creates consistent golden records for critical entities (and borrows DQM techniques such as cleansing); governance sets the policies, owners and standards that DQM enforces. All three sit under data management.

**Interview angle**
- *Q: Walk me through how you would introduce DQM on an existing pipeline.* — Profile the key tables to get a baseline, agree rules and thresholds with owners, add tests at ingestion and after each transformation, route failures to alerts with an owner, and track incidents and root causes so recurring problems are fixed at the source.

**Your experience hook:** Entity resolution with Splink, where blocking cut 14.7B candidate pairs to 77M, is the deduplication part of cleansing at scale.

---

## Data quality dimensions
*Source: [IBM — What Are Data Quality Dimensions?](https://www.ibm.com/think/topics/data-quality-dimensions)*

**In one line:** Data quality dimensions are a structured set of measurable aspects (most commonly accuracy, completeness, consistency, timeliness, validity and uniqueness) used to assess whether data can be trusted and used.

**Key points** (each dimension with a concrete check)
- **Accuracy** — does data match reality or a trusted source? *Check:* reconcile against the system of record, e.g. compare daily row counts and sums of `amount` between source and warehouse (control totals), or sample records against the source.
- **Completeness** — are required values and records present? *Check:* `not_null` tests on required columns, null-ratio thresholds, and row-count reconciliation or volume anomaly checks (today's rows vs. the recent average).
- **Consistency** — does the same fact agree across systems and with logic? *Check:* cross-table assertions (e.g. order total = sum of line items, department headcount ≤ company headcount), dbt `relationships` tests for foreign keys, and the same metric compared across two marts.
- **Timeliness** — is data available and current when needed? *Check:* `dbt source freshness` on a `loaded_at` column with `warn_after` and `error_after`, plus pipeline latency monitoring against an SLA.
- **Validity** — does data follow formats, types, ranges and business rules? *Check:* `accepted_values`, regex or range assertions (birth date not in the future, product code in the catalog), schema or type contracts at ingestion.
- **Uniqueness** — is each entity recorded once? *Check:* `unique` test on the primary key, or `unique_combination_of_columns` on a composite grain, and duplicate-rate monitoring after merges.
- The idea comes from Wang and Strong (1996), who listed 15 dimensions. There is no single standard, but 6–12 core dimensions are common. Extra ones include integrity, traceability, availability, reliability, precision and relevance. Implementation follows three steps: **assess and define** (profile, align stakeholders, set thresholds and rules), **measure and monitor** (metrics, dashboards, alerts across the lifecycle) and **correct and improve** (cleanse, fix collection, refine governance).

**Interview angle**
- *Q: Give me one concrete test for each quality dimension.* — Accuracy: source-to-target reconciliation of counts and sums. Completeness: `not_null` and volume checks. Consistency: `relationships` and cross-table assertions. Timeliness: source freshness against an SLA. Validity: `accepted_values` and range rules. Uniqueness: `unique` on the grain.

**Your experience hook:** The DataTalksClub Zoomcamp's dbt module uses exactly these generic tests (`unique`, `not_null`, `accepted_values`, `relationships`) on staging and fact models.

**Beyond IBM:** Different consumers weight dimensions differently (finance needs accuracy, a fraud model needs timeliness), so set thresholds per dataset and use case, not globally. Tests on every column create alert fatigue; prioritise primary keys, join keys and the columns behind key metrics.

---

## Data accuracy
*Source: [IBM — What Is Data Accuracy?](https://www.ibm.com/think/topics/data-accuracy)*

**In one line:** Data accuracy is how closely data reflects the true, real-world value it describes: correct, precise and free of errors.

**Key points**
- It is one core quality dimension, kept up through error correction, validation rules and governance (clear policies for collection, ownership, storage and use).
- Benefits: efficient operations, regulatory compliance (SOX, Basel III demand accurate financial data), trustworthy AI output and customer satisfaction, which matters most in healthcare, finance and manufacturing.
- Causes of inaccuracy: human error (the main one), system errors and bugs, outdated information, duplicates, incomplete data, inconsistent formats across systems, biased data and poor collection methods or tools.
- Measuring it means checking correctness through validation, verification and comparison with known sources of truth.
- Methods to maintain it: auditing, cleansing, profiling ("data archaeology"), validation, integration, observability and governance.

**Compare / don't confuse with:** *Accuracy vs. integrity* — accuracy is about whether individual values are correct; integrity is about keeping data accurate, complete and consistent across its lifecycle, including when it moves between systems.

**Interview angle**
- *Q: Tests pass but finance says the numbers are wrong. How?* — Most tests check form (nulls, types, uniqueness), not truth. Accuracy needs reconciliation against an independent source: compare totals per day with the ledger, check the join logic and filters, and look for late-arriving or duplicated records.

**Beyond IBM:** Accuracy is the hardest dimension to automate because you need a reference to compare with. In practice it comes from reconciliation jobs, control totals and periodic sampling against the system of record.

---

## Data integrity
*Source: [IBM — What is Data Integrity?](https://www.ibm.com/think/topics/data-integrity)*

**In one line:** Data integrity is the assurance that data stays accurate, complete and consistent throughout its lifecycle and is protected from loss, leaks, corruption and unauthorised change.

**Key points**
- It is like quality control for a product's raw materials, and it combines technology, policy and people. It depends on error checking, validation, encryption, access control and backups.
- Threats: human error, transfer errors, bugs, malware, hacking, compromised hardware and physical damage.
- Five types: **entity integrity** (each row uniquely identified, via primary keys), **referential integrity** (foreign keys point to existing rows, so no orphans), **domain integrity** (allowed types, formats and ranges per column, e.g. age between 1 and 120), **user-defined integrity** (custom business rules) and **physical integrity** (storage survives power loss, hardware failure and disasters).
- Practices: validate as close to the point of entry as possible, enforce constraints in the database, set retention rules, take regular geographically spread backups, control access, use redundant hardware and UPS, use lineage tools for audit trails, and educate staff.
- Regulated sectors make it explicit: FDA rules in pharma, FINRA in finance, KYC checks to verify customers.

**Compare / don't confuse with:** *Integrity vs. quality vs. security* — quality describes the condition of data (accuracy, completeness and so on); security protects it from unauthorised access; IBM frames integrity as the overarching principle that includes elements of both and adds protection against tampering.

**Interview angle**
- *Q: How do you enforce referential integrity in a cloud warehouse?* — Many warehouses (for example BigQuery and Snowflake) accept primary and foreign key declarations but do not enforce them. So you enforce them in the pipeline: dbt `unique`, `not_null` and `relationships` tests, plus merge logic that handles late-arriving dimensions (e.g. an "unknown" member row).

**Your experience hook:** The KYC/KYB work touched several integrity types at once: UUID surrogate keys for entity integrity, idempotent queue consumers so retries didn't corrupt state, and access control around PII.

---

## Data reliability
*Source: [IBM — What Is Data Reliability?](https://www.ibm.com/think/topics/data-reliability)*

**In one line:** Data reliability is how far data can be counted on to be consistent and free of errors over time and across sources.

**Key points**
- IBM measures it through three questions: is it **valid**, is it **complete**, is it **unique**? Some teams also track when and how a source changed, how often it updates, where it came from and how often it is used.
- Challenges: flawed or biased collection, inconsistency over time (changed definitions or systems), human error, a changing subject (consumer behaviour drifts), weak governance and stewardship, source changes and integration, and duplicates.
- Good practices: standardise collection, train data collectors, run regular audits with root-cause fixes, use tested tools (e.g. check that streams don't drop or duplicate events), clean rigorously, keep a data dictionary, make processing reproducible (version control for data and code), govern who can change data, and back up.
- Observability is the main lever: monitoring, alerting, lineage and SLA tracking catch problems before bad data spreads.

**Compare / don't confuse with:** *Reliability vs. quality* — quality is the broad set of criteria; reliability is narrower and is about whether data stays dependable over time and repeated use. *Reliability vs. validity* — reliable means consistent and repeatable; valid means it follows the rules and measures what it should. You can have one without the other.

**Interview angle**
- *Q: How do you make a streaming pipeline reliable?* — Assume at-least-once delivery: make consumers idempotent (dedup on an event ID or upsert on a key), store offsets or checkpoints so restarts resume cleanly, send poison messages to a dead-letter queue, and monitor lag and throughput so a silent stall is detected.

**Your experience hook:** RabbitMQ and Redis queues with idempotency keys, retries and dead-letter queues are direct applications of "test that events are not missed or duplicated".

**Beyond IBM:** In data engineering, reliability also means *pipeline* reliability: idempotent, re-runnable jobs, backfills that give the same result, and deterministic transformations. That is often what "reliable data" means in a job interview.

---

## Data observability
*Source: [IBM — What is Data Observability?](https://www.ibm.com/think/topics/data-observability)*

**In one line:** Data observability is the practice of continuously monitoring and understanding the health of data and pipelines across the stack, so that issues are detected, diagnosed and fixed in near real time.

**Key points**
- It goes beyond monitoring (which only says something is wrong) to triage, root-cause analysis, lineage and SLA tracking. IBM calls it the natural evolution of the data quality movement and an enabler of DataOps.
- Why: bad data fails silently. A broken dashboard is noticed at once; a wrong one may run for months. Example: Unity's 2022 ingestion of bad customer data cost an estimated USD 110M.
- Complexity drivers: more external sources (drift, schema changes, delays), more complex transformations and automated "modern data stack" pipelines that hide how data is delivered.
- DataOps cycle: **detection** (schema and row-level validation, reactive), **awareness** (centralised metadata and lineage so teams can trace issues upstream and impacts downstream), **iteration** (data-as-code, repeatable standards).
- **Five pillars:** freshness, distribution (values within expected ranges), volume (row counts versus expectations), schema (structure changes) and lineage (where it broke, what is affected downstream).
- Hierarchy: operational monitoring of data in motion (pipelines) and data at rest (datasets), then column-level profiling, then row-level validation. Starting at row level loses the big picture. Implementation: define metrics, choose tools, standardise libraries, instrument pipelines, store metrics in a time-series store, analyse, alert, connect to incident management, and review regularly.

**Compare / don't confuse with:** *Observability vs. data quality* — observability finds problems; it does not fix or prevent them, and you can have good quality without a formal observability programme. *Observability vs. governance* — governance sets the rules and ownership; observability monitors whether quality, availability and lineage hold up.

**Interview angle**
- *Q: What would you monitor on a critical table?* — The five pillars: freshness (max load timestamp vs. SLA), volume (row count vs. the recent baseline), schema (column and type changes), distribution (null rates, value ranges, category mix) and lineage so you know which dashboards and models are affected when something breaks.

**Your experience hook:** CDC streams that stalled silently, then fixed with resume tokens plus monitoring, is a textbook case for freshness and volume monitoring; Prometheus, Grafana and ELK are the instrumentation and alerting layers IBM's implementation steps describe.

**Beyond IBM:** Open-source options include dbt tests and source freshness, Great Expectations, Soda and Elementary; commercial platforms add ML-based anomaly detection. The "five pillars" framing was popularised by the data observability vendor community, not by IBM.

---

## Bad data
*Source: [IBM — What Is Bad Data?](https://www.ibm.com/think/topics/bad-data)*

**In one line:** Bad data is information that damages decision-making because it is inaccurate, incomplete, inconsistent, outdated, duplicated, invalid or biased.

**Key points**
- Its special danger is that it is quiet: unlike an outage, it can go unnoticed for years while decisions are made on it.
- Types follow the quality dimensions, plus **bias**: not a dimension itself, but skewed or unrepresentative data that leads to unfair or wrong outcomes, especially in ML. Intentional redundancy (replicas for availability) is not "duplicate data".
- Impact: missed opportunities, inefficiency, reputational damage, compliance penalties (SOX fines). Unity's ad algorithm, trained on bad customer data, had to be rebuilt.
- Causes: system failures and poor architecture, data decay, unreliable collection, weak governance, human error, failed integrations or migrations (mismatched formats, unnoticed dependencies).
- Prevention: governance and strategy; monitoring and visibility (lineage, audits, profiling, observability); cleansing and remediation (standardisation, dedup, outliers, missing values, validation, increasingly AI-assisted); and data literacy so people spot problems.

**Interview angle**
- *Q: How do you stop bad data reaching dashboards?* — Validate on the way in, and publish only after checks pass: write to a staging location, run tests, and swap or merge into the production table only on success. Failing records go to a quarantine table with an alert to the owner.

**Beyond IBM:** The "write-audit-publish" pattern (write to a hidden branch or staging table, audit, then publish atomically) is the standard practical answer; table formats like Iceberg support it with branches or snapshots.

---

## Stale data
*Source: [IBM — What Is Stale Data?](https://www.ibm.com/think/topics/stale-data)*

**In one line:** Stale data is information that is outdated, no longer matches current conditions or no longer fits its intended use.

**Key points**
- Staleness is caused by time, not by a collection error. Data goes stale gradually (customer drift) or suddenly (a crisis or new tariffs make history irrelevant).
- Causes: refresh cadence slower than the use case (weekly batch feeding a real-time decision), latency across multi-hop pipelines, old datasets never refreshed or archived, upstream schema or logic changes not passed downstream (unversioned APIs) and badly configured cache invalidation.
- It looks reliable while being wrong: systems keep running and decisions keep being made. Data can be complete and consistent and still stale.
- Risks: ML models learning outdated patterns, RAG retrieving outdated context and presenting it confidently, automated and agentic workflows acting on old facts, slowly worsening decisions, and compliance exposure (outdated personal data under GDPR).
- Detection: freshness SLAs; timestamps and update-cadence thresholds with alerts; lineage and provenance to locate the lag; anomaly detection (a metric that suddenly plateaus often means data stopped flowing).
- Prevention: choose batch, streaming or hybrid based on how fast the source changes; show last-updated metadata to consumers; automatic expiry, quarantine or archive rules; governance with freshness thresholds per use case and owners; observability across ingestion and transformation; and monitoring of model inputs at inference time, not just training data.

**Interview angle**
- *Q: The Airflow DAG is green but the dashboard shows yesterday's data. What happened?* — Success of the job doesn't prove fresh data: the upstream extract may have returned zero rows, a CDC stream may have stalled, or a cache wasn't invalidated. That's why freshness should be checked on the data itself (max event or load timestamp) as well as on task status.

**Your experience hook:** The silently stalled CDC streams were stale data in its purest form, and for the RAG systems, the same risk applies to the document index if re-indexing falls behind.

**Beyond IBM:** In dbt, declare `freshness` on sources with a `loaded_at_field` and `warn_after`/`error_after`, and run `dbt source freshness` before building models so stale inputs stop the build.

---

## Dark data
*Source: [IBM — What Is Dark Data?](https://www.ibm.com/think/topics/dark-data)*

**In one line:** Dark data is information an organisation collects and stores but never uses for analysis or decisions.

**Key points**
- It is common: in a Splunk survey, most respondents said half or more of their data was dark. Cheap storage encourages "keep everything just in case".
- It stays unused because metadata is missing, formats are unreadable by tools, or it can't be queried.
- Causes: no one knows it exists, silos, weak governance, legacy systems, incomplete integration, shifting priorities, limited resources or literacy, poor quality (so nobody trusts it), retention kept past the legal period, and ROT data (redundant, obsolete, trivial).
- It can be structured (logs, IoT, CRM, ERP), unstructured (emails, PDFs, call recordings, video) or semi-structured (HTML, XML, invoices).
- Costs: storage, privacy-law exposure (rules apply to unused data too), missed opportunities, slower search and analysis, and security and breach risk.
- Fixes: break down silos, classify and inventory all data, set governance with clear retain, archive or destroy rules, and use ML/AI to classify content and redact sensitive information. For quality-driven dark data: assess the current state, fix root causes, communicate progress.

**Interview angle**
- *Q: Is storing everything forever a good default?* — No. Unused data still costs money and is still in scope for privacy law and breaches. Keep raw data with a defined retention period and an owner, catalog it so it can be found, and delete or archive what has no purpose.

**Beyond IBM:** Object-storage lifecycle policies (move to cold tiers, then expire) and partition expiration in warehouses are the simplest technical controls; GDPR's storage-limitation principle makes this a legal issue, not only a cost one.

---

## Dirty data
*Source: [IBM — What Is Dirty Data?](https://www.ibm.com/think/topics/dirty-data)*

**In one line:** Dirty data is data that is inaccurate, invalid, incomplete or inconsistent, so it can't be relied on for business use.

**Key points**
- Forms: duplicates, nulls, inconsistent formats, outdated values, invalid entries, broken relationships between records and conflicting definitions across systems. It can enter at any point in the lifecycle.
- Costs: poor decisions, wasted marketing, fines and failed audits, time lost on reconciliation, dependence on IT, slow decisions and lower ROI from analytics and AI. Gartner expects many AI projects without AI-ready data to be abandoned.
- Root causes: human error, silos, weak governance, flawed integration (schema mismatches, partial transfers), technical debt in legacy systems, no validation at entry, priorities that reward speed over accuracy, and ML feedback loops where model outputs re-enter as inputs without checks.
- Eight cleaning steps: capture context and usage; define requirements and relationships; review samples; profile to set baselines (row counts, distributions, nulls, duplicates); document rules and constraints; analyse root causes; remediate and add preventive controls; track quality metrics over time.
- Tools range from integration platforms and matching/MDM suites to address and email validation services, observability tools and built-in tests in BI, ETL and transformation frameworks.
- Governance stops it coming back: a steering committee, data owners (accountable) and stewards (day-to-day), clear standards, and audits and monitoring.

**Compare / don't confuse with:** *Dirty vs. bad data* — IBM uses them almost interchangeably. "Dirty" stresses data that needs cleaning; "bad" is the wider term and includes bias and staleness.

**Interview angle**
- *Q: You find 3% of incoming records fail validation. Drop, fix or load?* — Don't silently drop. Load valid rows, route failures to a quarantine or dead-letter table with the reason, alert the owner, and fix the cause upstream. Whether the batch should fail depends on an agreed threshold and on how critical the table is.

**Your experience hook:** Splink entity resolution and the phone-number collisions fixed with surrogate keys were both clean-ups of duplicate and conflicting records that arose when several sources were combined.

---

## AI data quality
*Source: [IBM — Why AI Data Quality Is Key To AI Success](https://www.ibm.com/think/topics/ai-data-quality)*

**In one line:** AI data quality is how accurate, complete, reliable and fit for use data is across the AI lifecycle (training, validation and deployment), including AI-specific concerns like representativeness, bias, label accuracy and noise.

**Key points**
- Poor data is one of the most common reasons AI projects fail; models inherit and can amplify errors, gaps, staleness and bias. Only a small share of AI initiatives scale, and AI-first organisations are far more likely to have mature data governance.
- A few bad outputs can make people distrust the whole tool. There are legal and ethical stakes too: discrimination in lending or hiring, and laws like the EU AI Act that cover training-data quality and provenance.
- The dimensions are reframed for AI: **accuracy** includes label noise and proxy variables; **completeness** means covering edge cases, rare events and minority groups; **integrity** means full provenance and reproducible preparation; **consistency** means the same processing for historical and new data; **timeliness** means watching for data and concept drift; **relevance** means each feature actually helps the model's purpose.
- Practices: profile and explore early (lineage, outliers, missing values, structured and unstructured data); use observability to link data changes to model outcomes; use AI to check data (anomalies, distribution shifts, dedup); and close the loop with remediation such as retraining, new labelling guidelines, preprocessing fixes or collecting more data for under-represented cases.

**Interview angle**
- *Q: Your model's accuracy dropped in production. What data checks do you run?* — Compare live feature distributions with training (drift), check feature freshness and null rates, look for training/serving skew in how features are computed, check whether label definitions changed, and confirm the upstream sources didn't change schema or meaning.

**Your experience hook:** Building ML-ready datasets with Airflow and Redpanda, and credit-scoring features from MongoDB aggregations, puts you on the data side of exactly these problems; RAG systems add retrieval quality and index freshness.

**Beyond IBM:** Two classic traps the article doesn't name: *data leakage* (features using information not available at prediction time) and *point-in-time correctness* (joining features as they were when the event happened, not as they are now). Both are data engineering problems.

---

## Self-test

1. List the six core data quality dimensions and give one concrete automated check for each.
<details><summary>Answer</summary>Accuracy: reconcile row counts and sums against the source system. Completeness: `not_null` tests and volume checks. Consistency: `relationships` tests and cross-table assertions. Timeliness: source freshness against an SLA. Validity: `accepted_values`, range or regex rules. Uniqueness: `unique` test on the primary key or grain.</details>

2. How do data quality, data integrity and data profiling differ?
<details><summary>Answer</summary>Quality is the broad set of criteria for fitness for use. Integrity covers accuracy, completeness and consistency across the lifecycle, with protection against corruption and tampering. Profiling is the process of examining data's structure and content to measure quality and set a baseline.</details>

3. Name the five types of data integrity.
<details><summary>Answer</summary>Entity (unique primary keys), referential (valid foreign keys, no orphans), domain (allowed values, types and ranges), user-defined (custom business rules) and physical (protection of stored data against hardware failure, outages and disasters).</details>

4. Why are primary and foreign keys in a cloud warehouse often not enough to guarantee integrity?
<details><summary>Answer</summary>Warehouses such as BigQuery and Snowflake let you declare them but don't enforce them (apart from NOT NULL in Snowflake). Integrity has to be enforced with pipeline tests (unique, not_null, relationships) and careful merge logic.</details>

5. What are the five pillars of data observability?
<details><summary>Answer</summary>Freshness, distribution, volume, schema and lineage.</details>

6. What is the difference between data reliability and data validity?
<details><summary>Answer</summary>Reliability is consistency and repeatability over time and across measurements; validity is conformance to rules and formats and measuring what is intended. Data can be reliably collected but invalid, or valid but produced by an unreliable process.</details>

7. A pipeline reports success but the data is stale. Give three possible causes and how you would detect staleness.
<details><summary>Answer</summary>Causes: an upstream extract returned zero rows, a CDC stream stalled, a cache wasn't invalidated, or upstream logic changed without notice. Detect it by checking the data itself: max event or load timestamp versus a freshness threshold (e.g. dbt source freshness), volume anomaly checks and plateau detection on key metrics.</details>

8. What is dark data, and why is "store everything" risky?
<details><summary>Answer</summary>Data that is collected and stored but never used. Storing it costs money, it remains in scope for privacy regulation and breaches, and it makes useful data harder to find. Retention rules, cataloguing and deletion policies reduce the risk.</details>

9. How should a pipeline handle records that fail validation?
<details><summary>Answer</summary>Don't drop them silently. Route them to a quarantine or dead-letter table with the failure reason, alert the owner, decide via an agreed threshold whether the batch fails, and fix the root cause upstream. Write-audit-publish keeps bad data out of production tables.</details>

10. Name three ways the data quality dimensions are reframed for AI.
<details><summary>Answer</summary>Any three: accuracy includes label noise; completeness means coverage of edge cases and minority groups; integrity means provenance and reproducible preparation; consistency means the same processing for historical and new data; timeliness means monitoring data and concept drift; relevance means features genuinely support the model's purpose.</details>
