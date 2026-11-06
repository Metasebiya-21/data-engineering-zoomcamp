# 2. Data

> This section covers the raw material itself: what data is and how it is classified (quantitative/qualitative, structured/semi-structured/unstructured, metadata, big data), the three states of data and how to protect data while it moves, how to unify scattered data into one trusted view, how metadata is managed, and what makes a collection of data a usable dataset. Interviewers use these basics to test precise vocabulary and to lead into design questions: which store for which shape of data, how to secure a stream, how to merge sources without duplicates, and how to make data discoverable.

## Data
*Source: [IBM — What is Data?](https://www.ibm.com/think/topics/data)*

**In one line:** Data is any recorded fact, number, text, observation or signal that can be processed and analysed to support decisions.

**Key points**
- Common types (one item can be several at once): quantitative (measurable; discrete or continuous), qualitative (descriptive, such as reviews), structured, unstructured, semi-structured, metadata ("data about data") and big data (too large or complex for traditional systems).
- Uses listed: predictive analytics, generative AI, healthcare, research, cybersecurity (SIEM), operational efficiency, personalisation, open government data, and BI.
- Data collection starts with clear objectives and sources, then acquisition, cleaning and integration, with ongoing quality checks. Typical sources: transactions, web analytics, IoT, surveys, social media, public/open data.
- Storage foundation: lakes (cheap, raw), warehouses (structured, for analysis), lakehouses (both).
- Roles: data scientists build models and algorithms for longer-term questions; data analysts answer specific business questions, often with dashboards.
- Data protection has two parts: data security (stopping unauthorised access, corruption or theft) and data privacy (the person's control over how their data is used). Threats: insiders, phishing/social engineering, ransomware, cloud misconfiguration. Controls: encryption, backups, firewalls, MFA, IAM, anti-malware, DLP.
- For AI, training-data quality and diversity drive model quality; biased or incomplete data gives biased output.

**Compare / don't confuse with:** Data security (protecting data from unauthorised access) vs. data privacy (the individual's right to control use of their personal data). You can have secure data that is still used in a way that breaks privacy rules.

**Interview angle**
- *Q: What is the difference between data security and data privacy?* — Security is the technical and organisational protection of data against access, loss or tampering; privacy is about lawful and consented use of personal data. Encryption helps security; purpose limitation and consent are privacy.

**Your experience hook:** PII pseudonymisation and access control on the KYC/KYB platform cover both halves: access control is security, pseudonymisation limits privacy exposure.

**Beyond IBM:** Pseudonymised data is still personal data under GDPR because it can be re-linked with the key; only truly anonymised data falls outside it.

---

## Structured vs. unstructured data
*Source: [IBM — Structured vs. Unstructured Data: What's the Difference?](https://www.ibm.com/think/topics/structured-vs-unstructured-data)*

**In one line:** Structured data follows a fixed schema of rows and columns; unstructured data has no predefined model; semi-structured data sits between, carrying its own tags or keys.

**Key points**
- Four differences: format (predefined model or none), storage (relational DBs/warehouses vs. native format in NoSQL stores or lakes), use cases (classic ML and BI vs. NLP and generative AI), complexity (easy for business users vs. needs specialist skills).
- Structured: queried with SQL in an RDBMS; used for CRM, BI, inventory, model training. Pros: easy to use, works well with ML, many tools. Cons: built for one purpose, rigid schemas that are costly to change.
- Unstructured: emails, documents, chat, call transcripts, images, video, sensor data; IBM puts it at about 90% of enterprise data and growing faster than structured data. Used for RAG, gen AI, sentiment analysis.
- Unstructured pros: flexible, quick to collect, cheap to store (file systems, lakes). Cons: needs expertise and specialised tools, and has more quality problems.
- Semi-structured: uses tags or markers to organise records (JSON, XML; email headers around a free-text body). Useful for web scraping and integration.

**Compare / don't confuse with:** Schema-on-write (structure enforced when data is stored, as in a relational table) vs. schema-on-read (raw data stored as-is and structure applied when queried, as in a lake).

**Interview angle**
- *Q: How would you store and query JSON events from many services?* — Land them raw (lake or document store) for flexibility, then parse the stable fields into typed columns in a warehouse/lakehouse table for analytics, keeping the raw payload for reprocessing.

**Your experience hook:** Your MongoDB → Oracle incremental ETL is a semi-structured-to-structured conversion: flexible documents mapped into a fixed relational schema, including updates and deletes.

**Beyond IBM:** IBM lists CSV as semi-structured; most engineers treat CSV as structured tabular data without enforced types. Columnar formats such as Parquet are the usual way to store structured data in a lake.

---

## Data in motion
*Source: [IBM — What Is Data in Motion?](https://www.ibm.com/think/topics/data-in-motion)*

**In one line:** Data in motion (data in transit) is data actively moving between systems, networks or devices, one of three data states alongside data at rest and data in use.

**Key points**
- Three states, using a spreadsheet: at rest (stored in a folder), in use (open and being edited), in motion (emailed or synced).
- Mechanics: data is split into packets, routed and reassembled. Event-driven systems react as events happen; Kafka routes high-throughput streams; Flink transforms and analyses them in flight; APIs standardise exchange.
- Why now: fraud detection, patient monitoring, dynamic pricing and AI agents need fresh data, and scheduled batch introduces lag.
- Securing it: encryption (AES-256 for symmetric encryption, TLS 1.3 for channels, message-level signing/encryption), RBAC and MFA, integrity checks (hashes, schema validation), and regulations (GDPR, PCI DSS for cardholder data on public networks, HIPAA).
- Types: across networks, to and from endpoints (needs disk encryption, DLP), and between cloud services.
- Practices: choose ingestion by how fast sources change and what freshness is needed; extend governance to streams (freshness SLAs, schema enforcement at ingestion, lineage, access at each hop); observe ingestion rate, latency and quality; check agent pipelines for hidden batch steps.

**Compare / don't confuse with:** Data at rest vs. in motion vs. in use. Also, "data in transit" in the security sense (protecting bytes on the wire) vs. streaming data processing (continuous event pipelines); IBM's article blends the two.

**Interview angle**
- *Q: How do you secure a Kafka-based pipeline?* — TLS on all connections, authentication (SASL or mTLS) and ACLs per topic, encryption at rest on brokers, schema validation at the producer or ingestion layer, and masking of PII before it reaches broad-access topics.
- *Q: Is streaming always better than batch?* — No; it costs more and is harder to run. Use it where latency changes the outcome (fraud, alerts), batch elsewhere.

**Your experience hook:** Your Airflow + Redpanda pipelines and the CDC streams you hardened with resume tokens and monitoring are real data-in-motion systems, and in fintech they fall under the same encryption and access duties.

**Beyond IBM:** Stream reliability depends on delivery semantics (at-most-once, at-least-once, exactly-once) and on idempotent consumers, because retries and replays will deliver duplicates.

---

## Unified data
*Source: [IBM — What Is Unified Data?](https://www.ibm.com/think/topics/unified-data)*

**In one line:** Unified data is data from many separate sources combined into one consistent, trusted view, either by moving it into one place or by connecting it where it lives.

**Key points**
- Problem: data is fragmented across mainframes, clouds, lakes, CRMs and tools, with each team using different formats and rules, giving inconsistent and low-quality results.
- For AI: a RAG chatbot over HR documents only gives correct answers if it can reach every region's current policies; fragmented data means partial or outdated answers.
- Benefits: lower cost (less tool sprawl and duplicate storage), wider access and self-service, less data wrangling for engineers, faster decisions, and simpler security and compliance.
- Architectures: data fabric (virtual unified view via active metadata), data mesh (domain ownership), lakehouse, unified data platform (one interface over many sources).
- Integration methods beyond ETL: zero-copy integration (query data at source without copying), data virtualisation (an abstraction layer), real-time integration.
- Governance for unification: MDM, identity resolution and semantic layers for one trusted view; quality steps (cleansing, validation, enrichment, deduplication, normalisation); access controls; catalogues and observability.
- Non-technical factors: team culture, skills shortages, compatibility with existing systems, privacy at every stage.

**Compare / don't confuse with:** Physical consolidation (copy into a warehouse/lakehouse) vs. virtualisation/zero-copy (leave data in place and query through a layer).

**Interview angle**
- *Q: How would you merge customer records from three source systems?* — Assign your own surrogate keys rather than reusing source IDs, standardise fields, run deterministic then probabilistic matching (entity resolution) to link duplicates, build a golden record with survivorship rules, and keep lineage back to each source row.

**Your experience hook:** Consolidating 2M+ records from multiple source databases with CDC, using UUID surrogate keys to fix ID and phone-number collisions, and resolving entities with Splink is exactly data unification with identity resolution.

**Beyond IBM:** Virtualisation and zero-copy avoid copies but push query load and latency onto the source systems and depend on their uptime; copying into a lakehouse costs storage but isolates analytics from production.

---

## Metadata management
*Source: [IBM — What Is Metadata Management?](https://www.ibm.com/think/topics/metadata-management)*

**In one line:** Metadata management is the organised capture, standardisation and use of "data about data" so that data can be found, understood, trusted and governed.

**Key points**
- Analogy: metadata is the label on a book; metadata management is the library's system for organising books by those labels.
- Supports governance (stewards and standards), integration (consistent definitions across sources), lineage (impact analysis, compliance, AI explainability), quality (completeness, accuracy, consistency, freshness), accessibility (search, self-service) and security (access to sensitive metadata).
- Five types: descriptive (title, author, keywords), structural (how parts relate), administrative (owner, permissions, retention), technical (format, encoding, location), preservation (long-term usability, backups, migrations).
- Standards: Dublin Core, ISO/IEC 11179, FAIR principles; DCAT and PREMIS for web/open data and preservation; industry ones such as HL7/FHIR (health), ISO 20022 (finance), ISO 19115 (geospatial).
- Tool categories: standalone catalogues, metadata-aware ETL, enterprise governance suites, cloud-native catalogues, open-source tools.
- Challenges: scale, inconsistent naming across systems ("Customer ID" vs. "Client Code"), sensitive content in metadata, and teams that will not document.
- Trends: active metadata (triggers alerts and actions), AI auto-classification and knowledge graphs, metadata as the backbone of fabric and mesh.

**Compare / don't confuse with:** Passive metadata (a static catalogue someone has to read) vs. active metadata (metadata that drives automation, such as alerts when a schema changes or policies applied by tag).

**Interview angle**
- *Q: How do you use lineage in practice?* — Before changing or dropping a column, check lineage to see which models, dashboards and ML features depend on it; after an incident, trace back to the source of bad data.

**Beyond IBM:** Common open-source catalogues are DataHub, OpenMetadata and Amundsen; dbt also generates lineage and documentation from the models themselves, and lakehouse catalogues (for example Unity Catalog or Apache Polaris for Iceberg) hold technical metadata and permissions.

---

## Dataset
*Source: [IBM — What is a Dataset?](https://www.ibm.com/think/topics/dataset)*

**In one line:** A dataset is an organised collection of related data, with a defined structure and context, prepared for analysis or for training models.

**Key points**
- Three defining traits: variables (the attributes measured), a schema (structure and relationships, tabular or nested JSON) and metadata (origin, purpose, usage rules).
- Not a dataset: a random pile of unrelated data points. APIs, databases and spreadsheets can hold or deliver datasets but are not datasets themselves.
- Types: structured (tables for CRM, inventory, finance), unstructured (text, images, audio, transcripts; key for AI), semi-structured (JSON, XML, HTML, logs mixing fields and free text). They are often combined.
- Sources: proprietary or public repositories, databases, APIs (often via automated Python pipelines), public platforms such as Data.gov or NYC Open Data.
- Uses: AI/ML (NLP, computer vision, predictive regression models, research), analysis (patterns, visualisation, statistics, hypothesis tests), and BI (real-time KPIs, customer behaviour, time series, supply chain).
- Considerations: quality (consistent formats, deduplication), interoperability (aligning CSV and JSON into a common schema), ethics and bias (PII, anonymisation, biased training data), and ongoing dataset management under GDPR/CCPA.

**Compare / don't confuse with:** Dataset (an organised, documented collection for a purpose) vs. database (a system that stores and serves data, which may contain many datasets).

**Interview angle**
- *Q: What makes a dataset "analysis-ready"?* — A stable, documented schema with clear types and grain, deduplicated and validated rows, known freshness, lineage to its sources, and PII handled according to who will use it.

**Your experience hook:** Your current pipelines produce analysis-ready and ML datasets, and your credit-scoring work (factor analysis and PCA on MongoDB aggregations) is a direct consumer of such datasets.

**Beyond IBM:** For ML, version the dataset (for example table snapshots/time travel or a tool like DVC) so each model can be reproduced, and state the grain (one row per what) explicitly; unclear grain is a common source of double counting.

---

## Self-test

1. What is the difference between quantitative and qualitative data, and can qualitative data be structured?
<details><summary>Answer</summary>Quantitative data is numeric and measurable (discrete or continuous); qualitative data is descriptive. Yes, qualitative data can be structured, for example coded survey answers or category labels.</details>

2. Distinguish data security from data privacy.
<details><summary>Answer</summary>Security protects data from unauthorised access, corruption or theft (encryption, IAM, backups). Privacy concerns an individual's control over how their personal data is collected and used (consent, purpose limits, rights).</details>

3. Give two examples each of structured, semi-structured and unstructured data.
<details><summary>Answer</summary>Structured: relational table of customers, financial report rows. Semi-structured: JSON API responses, XML files, log lines with fields. Unstructured: emails, PDFs, images, call recordings.</details>

4. What is schema-on-write vs. schema-on-read?
<details><summary>Answer</summary>Schema-on-write enforces structure when data is stored (relational tables, warehouses). Schema-on-read stores raw data and applies structure at query time (data lakes), trading upfront validation for flexibility.</details>

5. Name the three states of data and one protection for each.
<details><summary>Answer</summary>At rest (disk/database encryption, access control), in motion (TLS, authentication, integrity checks), in use (access control, masking, least privilege on the processing environment).</details>

6. Why do streaming consumers need to be idempotent?
<details><summary>Answer</summary>Retries, restarts and replays mean the same event can be delivered more than once under at-least-once delivery; an idempotent consumer produces the same result no matter how many times it processes the event.</details>

7. What are the trade-offs of data virtualisation / zero-copy integration versus copying data into a lakehouse?
<details><summary>Answer</summary>Virtualisation avoids duplicate storage and staleness from copies, but adds load and latency on source systems and depends on their availability. Copying costs storage and needs pipelines, but isolates analytics from production and allows history and heavy transformations.</details>

8. List the five metadata types IBM describes.
<details><summary>Answer</summary>Descriptive, structural, administrative, technical and preservation.</details>

9. What is the difference between passive and active metadata?
<details><summary>Answer</summary>Passive metadata sits in a catalogue for people to look up. Active metadata is used by systems to act automatically, such as raising an alert on schema change or applying an access policy based on a PII tag.</details>

10. What three characteristics define a dataset, and why is a database not itself a dataset?
<details><summary>Answer</summary>Variables, a schema and metadata. A database is a storage and retrieval system that can contain one or many datasets; the dataset is the organised, purposeful collection inside it.</details>
