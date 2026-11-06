# 11. Enterprise data management

> This section zooms out from pipelines and storage to the organisational layer: who owns data strategy, how data is shared and made usable across a company, how it is kept cheap and modern, and what goes wrong when it splinters into silos. In interviews, especially for a lead role, these topics show up as "how would you set up data at a company like ours?", "how do you agree expectations with consumers?" (data SLAs) and "how do you deal with three teams reporting three different revenue numbers?" (silos and fragmentation). You are expected to connect technical choices to business value and to people.

## Enterprise data management (EDM)
*Source: [IBM — What Is Enterprise Data Management?](https://www.ibm.com/think/topics/enterprise-data-management)*

**In one line:** EDM is the company-wide practice of organising, governing and optimising data across its whole lifecycle, from creation to archiving or deletion, so it stays accurate, accessible, secure and aligned with business goals.

**Key points**
- The driver is growth plus complexity: more data, more sources, real-time demands and privacy rules. Without EDM, silos persist, quality drops and integration gets expensive.
- AI raises the stakes: AI projects stall without clean, well-governed, "ready" data, and piecemeal tech investments make that harder.
- Four components: **data governance** (rules, roles, ownership, access control), **metadata management** (context, lineage, traceability), **technical enablers** (integration, ETL, MDM, profiling, cleansing, lineage tools) and **data lifecycle management** (retention, archiving, disposal in line with law and business needs).
- Benefits listed: managing complex data flows, better BI, support for innovation, protection of sensitive data and privacy compliance (audit trails, access controls for GDPR).
- AI can automate classification, cleansing, metadata tagging and anomaly detection inside an EDM programme.

**Compare / don't confuse with:** *Master data management (MDM)* is a subset of EDM. MDM keeps core entities (customer, product, supplier, employee) consistent and deduplicated across systems. EDM is the wider framework (governance, standards, architecture) that MDM needs in order to succeed.

**Interview angle**
- *Q: How does MDM relate to EDM?* — EDM sets the policies, ownership and architecture for all data; MDM works inside that framework to produce one trusted "golden record" per core entity. MDM without EDM tends to fail on unclear ownership and inconsistent policies.

**Your experience hook:** Consolidating 2M+ KYC/KYB records from several source databases with CDC, and introducing UUID surrogate keys to fix ID and phone-number collisions, is a hands-on MDM problem inside a wider data management effort.

**Beyond IBM:** In a small company "EDM" usually means a few concrete habits: a single warehouse or lakehouse, a catalog or at least documented models, named owners per dataset, retention rules and automated tests. You don't need a big programme to get most of the value.

---

## Chief data officer (CDO)
*Source: [IBM — What Is a Chief Data Officer (CDO)?](https://www.ibm.com/think/topics/chief-data-officer)*

**In one line:** A CDO is the executive accountable for getting maximum business value from company data, usually owning data strategy, governance, quality, analytics and security.

**Key points**
- The role started in the early 2000s focused on governance and compliance (for example Sarbanes-Oxley). It has since moved towards strategy and business outcomes.
- Typical responsibilities: governance framework, data security (often with the CISO), data lifecycle policies (including when to delete), analytics enablement and the data strategy itself.
- CDOs often start by building data literacy, then put policies in place for sharing and acting on insights.
- They use KPIs to show value (revenue growth, retention, customer satisfaction, conversion rates).
- Strong CDOs keep a clear line from data to business value, tie data work to wider digital investment, treat data as a driver of business-model innovation and work closely with partners.
- The AI era adds pressure to source trusted training data while keeping privacy and compliance. Some expect the title to grow into a chief data, analytics and AI officer (CDAIO).

**Compare / don't confuse with:** *CIO vs. CDO* — the CIO owns the technology infrastructure that processes data; the CDO owns how the data is used to create value. A *CAO* (chief analytics officer) may own analytics separately; a *CDAO* combines both.

**Interview angle**
- *Q: As a lead data engineer, how would you work with a CDO?* — Turn their strategy into a platform roadmap, make the cost and reliability trade-offs visible, and report on measurable outcomes like freshness SLAs, incident counts and adoption, not just tasks shipped.

**Beyond IBM:** Small companies rarely have a CDO. The CTO or a head of data covers the role, and a lead data engineer often ends up shaping strategy, governance and quality in practice.

---

## Data democratization
*Source: [IBM — Data Democratization Strategy for Business Decisions](https://www.ibm.com/think/topics/data-democratization)*

**In one line:** Data democratization means making data easy for anyone in the organisation to find, access, understand and use, whatever their technical background, without going through gatekeepers.

**Key points**
- It is more than opening access: it needs data literacy, trust in accuracy, and a governance approach covering training, policy and security.
- It differs from *data transparency* (accuracy and easy access wherever data lives). Democratization is about simplifying every data process, from storage to security.
- Architecture moves away from static "data at rest" towards distributed, real-time designs. IBM highlights **data fabric** (metadata-driven integration layer across sources) and **data mesh** (domain-owned data products). The two can complement each other.
- Benefits: productivity (fewer bottlenecks), more deliberate security and governance, easier tools for non-technical users. Risk: a poorly governed lake turns into a **data swamp**.
- Getting started: set business and data goals, audit current data, design a target framework, set controls (governance), integrate data, train people.
- For AI: pick use cases, find suitable datasets, use MLOps to scale, keep the pipeline from data to model visible.

**Interview angle**
- *Q: How do you give business users self-service without creating chaos?* — Curate a governed layer (tested, documented marts with clear owners and definitions), control access by role, and keep raw data away from casual users. Self-service on top of tested models avoids the "five versions of revenue" problem.

**Beyond IBM:** In practice self-service lives or dies on a shared semantic or metrics layer (for example dbt models plus a metrics definition) so that "active customer" means the same thing in every dashboard.

---

## Data literacy
*Source: [IBM — What Is Data Literacy?](https://www.ibm.com/think/topics/data-literacy)*

**In one line:** Data literacy is "the ability to read, understand, use and communicate with data for better decision-making."

**Key points**
- It is needed at every level, not just by data scientists, and now includes knowing how to question AI tools and judge their output.
- MIT's four core abilities: **read** data, **work with** data, **analyse** data and **argue with** data. Data storytelling (using numbers and visuals to persuade) is a key skill.
- People should know where a dataset comes from, how it fits the business context, and how to read a chart without being misled.
- Benefits: fewer silos, better privacy and security awareness, lower costs (technical teams freed from basic requests) and better decisions.
- Four principles of a data-literate culture: democratised access, organised and explainable data (lineage, shared definitions), trained "data citizens" who act as responsible stewards, and leaders who model and champion it.
- Lack of executive support is a common reason literacy programmes fail.

**Interview angle**
- *Q: What can a data engineer do to raise data literacy?* — Document models and columns, publish lineage, expose freshness and test status next to dashboards, and agree shared metric definitions. Users who can see where a number comes from and how fresh it is trust it more and misuse it less.

---

## Data marketplace
*Source: [IBM — What is a Data Marketplace?](https://www.ibm.com/think/topics/data-marketplace)*

**In one line:** A data marketplace is a platform where data providers list datasets or data products and consumers find, compare and acquire them through a self-service interface.

**Key points**
- Most are external (buying and selling data), but **internal marketplaces** let teams discover and reuse governed, business-ready data products, which helps break down silos.
- Key requirements: quality checks, strong security and privacy compliance (GDPR, CCPA), good search, and secure transactions with clear licences.
- Ecosystem roles: data providers, data consumers, platform operators and governance or regulatory authorities.
- How it works: data productization (packaging data as a product), discovery with rich metadata, API-based access, monetisation (payments, licences) and integration with other systems.
- Types: public, private or personal (people paid for their own data), hybrid, multilayered (raw, processed and derived tiers), B2B, IoT sensor and open data (for example data.gov).
- How a buyer should approach one: define needs, avoid vendor lock-in, choose reputable platforms and providers, check provenance and compliance, assess quality, govern the data, validate before integrating, and monitor over time.

**Interview angle**
- *Q: What would you check before using a purchased third-party dataset?* — Provenance and licence terms, legal basis for any personal data, quality (accuracy, completeness, freshness), update cadence and delivery SLA, and how it joins to our keys. Then validate it against our own data in a pilot before wiring it into production.

**Beyond IBM:** Real examples include cloud warehouse marketplaces and open sharing protocols that let consumers query a provider's data without copying files. Internally, a data catalog with owned, documented data products and data contracts does most of what an "internal marketplace" promises.

---

## Data optimization
*Source: [IBM — What Is Data Optimization?](https://www.ibm.com/think/topics/data-optimization)*

**In one line:** Data optimization is improving how data is organised and how good it is, so that storing, processing and analysing it is efficient and cheap.

**Key points**
- Drivers: huge volumes, varied formats (lots of unstructured data), pressure to have AI-ready data, and budgets (many firms overspend on cloud storage).
- **Storage:** compression, tiered storage (hot, cool, cold) and choosing between object, file and block storage.
- **Transformation and cleaning:** standardisation, deduplication, handling missing values and validation (type, format, range, uniqueness, consistency checks).
- **Metadata management:** descriptive (search), administrative (ownership, permissions, retention) and preservation (long-term usability).
- **Query optimisation:** filter early, add indexes, cache repeated results, partition tables and choose the right engine for the workload.
- Other techniques: parallel processing, access control, visualisation. Tools: cleaning, observability, governance, hybrid cloud and lakehouses.
- Governance supports optimisation, for example through quality metrics and retention policies that limit what you store.

**Interview angle**
- *Q: A warehouse bill doubled. Where do you look?* — Find the most expensive queries and tables, then check for missing partition filters or clustering, full-table rebuilds that could be incremental, repeated identical queries that could be cached or materialised, and data that could move to a colder tier or be deleted under a retention policy.

**Your experience hook:** Tuning PostgreSQL to sustain about 900 requests per second is query and storage optimisation in practice (indexes, query plans, connection handling).

**Beyond IBM:** In analytical stacks the biggest wins are usually columnar formats (Parquet), partition pruning and clustering (as in BigQuery, which bills by bytes scanned), and incremental models instead of full refreshes.

---

## Data modernization
*Source: [IBM — What is Data Modernization?](https://www.ibm.com/think/topics/data-modernization)*

**In one line:** Data modernization is moving data and data practices off legacy, siloed systems onto modern (usually cloud) platforms, while improving governance, security and accessibility.

**Key points**
- It covers migration, integration, cleansing, consolidation and transformation, and IBM treats it as largely the same thing as cloud migration.
- It goes together with stronger governance (quality, lineage, compliance), security (encryption, access control) and democratization (self-service on curated data).
- AI helps by automating analysis and anomaly detection. Cloud provides elastic storage and compute, real-time processing and managed security.
- Components: data integration, cleansing and quality, warehousing, advanced analytics and BI, cloud and big data tech, governance and security.
- Benefits: better decisions, efficiency, customer experience and compliance (audit trails, traceability).
- Use cases span finance (fraud, risk), healthcare, government, manufacturing (predictive maintenance), retail and telecoms (churn).

**Interview angle**
- *Q: How would you migrate a legacy reporting database to a cloud warehouse with low risk?* — Do an initial bulk load, then keep the target in sync with CDC or incremental loads. Run old and new side by side, reconcile row counts and key totals, switch consumers over gradually, and only then retire the legacy system.

**Your experience hook:** The MongoDB to Oracle pipeline with an initial-load mode, change-stream incremental sync (inserts, updates, deletes) and a full-refresh fallback follows the standard pattern for migrating data without downtime.

**Beyond IBM:** "Lift and shift" moves the same schemas and jobs to the cloud and rarely delivers the promised gains; re-modelling (for example ELT into a warehouse or an open table format like Iceberg or Delta) costs more upfront but removes the legacy constraints.

---

## Data reduction
*Source: [IBM — What Is Data Reduction?](https://www.ibm.com/think/topics/data-reduction)*

**In one line:** Data reduction is shrinking the amount of data stored or processed, by removing redundancy or representing it more compactly, ideally without losing useful information.

**Key points**
- Reduction does not have to mean information loss. Often the data is just stored more cleverly.
- It is wider than deduplication. It combines deduplication, consolidation and other techniques.
- Benefits: lower storage costs, and smaller, cleaner inputs for analytics, data mining and AI.
- **Dimensionality reduction:** fewer attributes, less noise and sparsity (for example PCA, feature extraction, wavelet transforms).
- **Numerosity reduction:** replace raw data with a smaller representation: parametric (regression, log-linear models) or non-parametric (histograms).
- **Data cube aggregation** (pre-aggregated multidimensional summaries), **discretisation** (binning continuous values into intervals) and **compression**: lossless (exact restore) vs. lossy (approximate).
- Preprocessing (cleaning, binning) usually comes before reduction.

**Compare / don't confuse with:** *Lossless vs. lossy compression* — lossless (gzip, Snappy, ZSTD in Parquet) restores the exact data; lossy (JPEG-style) does not, so it is fine for images but never for financial records.

**Interview angle**
- *Q: Name ways to reduce data volume in an analytics pipeline.* — Columnar compression, dropping unused columns early, pre-aggregating to the grain the business needs, sampling for exploration, and retention rules that expire raw data after a set period.

**Your experience hook:** Using PCA in the credit-scoring work is dimensionality reduction: it turns many correlated factors into a few components.

**Beyond IBM:** The article mixes two different ideas: storage reduction (compression, dedup, retention) and the data-mining textbook view of reducing data before modelling (PCA, histograms, data cubes). In interviews, say which one you mean.

---

## Data silos
*Source: [IBM — What Are Data Silos?](https://www.ibm.com/think/topics/data-silos)*

**In one line:** Data silos are isolated collections of data that one team or system holds and others cannot easily see or use.

**Key points**
- Causes: org structure (each team picks its own tools), IT complexity and legacy systems, a culture of "our data", limited budget or skills, regulation (for example storing data separately by region) and growth through mergers.
- Problems: manual reconciliation, duplicate storage, partial views leading to poor decisions, inconsistent quality, blocked innovation and AI, fragmented customer experience, and compliance controls repeated per silo.
- Warning signs: duplicated or conflicting data, slow manual data gathering, departments reporting different numbers, systems that can't talk to each other, different definitions for the same metric.
- Fixes: modernise data management (ETL/ELT, lakes, warehouses, lakehouses, streaming and replication for real-time sync), with open-source tools like Kafka, Spark, PostgreSQL and Airflow.
- Governance frameworks for safe sharing (role-based access, shared quality standards, automated compliance controls).
- Culture: cross-functional teams, named owners and stewards, data literacy, shared protocols, centres of excellence. The payoff is a single source of truth, efficiency, better decisions, consistent security and a unified customer view.

**Interview angle**
- *Q: Sales and finance report different revenue. What do you do?* — Trace both numbers back to their sources and definitions (lineage), agree a single definition with both owners, build one tested model for it, and point both dashboards at that model. The fix is part technical and part agreement between people.

**Your experience hook:** The KYC/KYB platform consolidated 2M+ records from multiple source databases through CDC, which is breaking down silos with real-time replication.

**Beyond IBM:** Not every silo is a bug. Data residency, PII isolation and tenant separation are deliberate boundaries. The goal is governed, intentional sharing, not one big copy of everything.

---

## Data strategy
*Source: [IBM — What Is Data Strategy?](https://www.ibm.com/think/topics/data-strategy)*

**In one line:** A data strategy is a plan for how an organisation will use data to improve decisions and processes and reach its business goals.

**Key points**
- It ties together collection, management, governance, analytics, quality and security. It is often owned by a CDO, or built jointly by executives and data teams.
- Six components: understand business goals and choose use cases, assess obstacles (technical and human), build a **roadmap** (goals, tech, people, timeline, success metrics), set **controls** (governance policies, shared terminology, regular reviews), aim for **small early wins**, then **scale** (progress reports, training, hiring).
- Disciplines it pulls together: data management, governance, integration, architecture, analytics, security and quality.
- Why it matters: better decisions, a base for AI, productivity, lower cost (efficient storage, fewer breaches and fines), innovation and competitive edge.
- Challenges: low data maturity, poor quality, silos, missing governance and a culture that isn't aligned.

**Interview angle**
- *Q: You join as the first lead data engineer. What are your first 90 days?* — Talk to stakeholders to find the two or three decisions that matter most, audit current sources and pain points, ship one small high-value use case end-to-end with tests and an owner, then turn the lessons into a roadmap with clear metrics.

**Beyond IBM:** A strategy that does not name owners, a budget and what you will *not* do is just a list of wishes. Start with use cases, not tools.

---

## Data SLA
*Source: [IBM — What's a data Service Level Agreement (SLA)?](https://www.ibm.com/think/topics/data-sla)*

**In one line:** A data SLA is a written, public promise from a data team to its consumers to deliver data of a defined quality within defined parameters, such as freshness by a set time.

**Key points**
- Unwritten expectations ("in a reasonable time") mean different things to different people. Writing them down removes the ambiguity and builds trust.
- A short template (250–500 words) with six parts: **purpose**, **promise**, **measurement** (what, who, time frame), **ramifications** of missing it, **requirements** from consumers (it is two-way) and **signatures**.
- Example promise: sales data at a quality score of at least 95% by 05:00 every day, measured by run start and end times, record counts, null ratios and distribution or drift scores.
- To hit it: define "good data", use observability to know when pipelines are down, track time to detection, time to resolution and incidents per asset, and record root causes (late upstream delivery, timeouts, stuck queues, permission errors, schema changes).
- Be careful what you promise: 99.999% uptime is about five minutes of downtime a year and needs on-call staff and redundancy.
- Operational practices: ticketing, named owners (with cover), alerts to chat or paging tools, a written incident-response plan with an incident commander, status banners on dashboards, consumer surveys, blameless post-mortems, and publishing the SLA where everyone can see it.

**Compare / don't confuse with:** *SLA vs. SLO vs. SLI* — the SLI is the measured indicator (for example, hours since last load), the SLO is the internal target (fewer than 2 hours, 99% of days), and the SLA is the agreement with consumers, often looser than the SLO, that says what happens if it's missed.

**Interview angle**
- *Q: How would you define and enforce a freshness SLA for a daily table?* — Agree the deadline and consequences with consumers, measure the max `loaded_at` timestamp against it with a freshness check (such as dbt source freshness) or a metric, alert before the deadline is breached, and review misses in post-mortems.

**Your experience hook:** Adding resume tokens and monitoring after CDC streams stalled silently is exactly the gap a freshness SLA closes: a job that "runs" but delivers nothing has to be caught by measuring the data, not the process.

---

## Data fragmentation
*Source: [IBM — What Is Data Fragmentation?](https://www.ibm.com/think/topics/data-fragmentation)*

**In one line:** Data fragmentation is when an organisation's data is scattered, without control, across many systems, clouds, databases and documents, so it is hard to access, govern and use.

**Key points**
- Symptoms: no single source of truth, a lot of manual searching and reconciling, slow decisions on stale data, rising IT costs (redundant tools and storage) and security or governance blind spots.
- It blocks enterprise AI: execution slows, models see only part of the picture, and inconsistent governance raises risk. RAG in particular needs broad access to proprietary data.
- Causes: hybrid multicloud estates, disconnected tools per team ("SaaS sprawl"), growing unstructured data volumes and weak governance (each team invents its own standards).
- Fixing it does not mean putting everything in one store. The goal is to connect the right data to the right people at the right time.
- Strategies: change the culture (data as a product), stronger governance, consolidating platforms (warehouse or lake via ETL/ELT), integration (ETL/ELT, replication, zero-copy or federated queries, iPaaS), data fabric, and AI/ML for discovery, classification and lineage.
- The term also has a second, unrelated meaning: deliberate fragmentation (partitioning) of data in a DBMS or file system for performance.

**Compare / don't confuse with:** *Fragmentation vs. silos* — a silo is one isolated store tied to one team; fragmentation is the overall state of data being spread across many places, often with overlapping copies.

**Interview angle**
- *Q: When would you query data in place instead of copying it into the warehouse?* — Query in place (federation, zero-copy) for occasional or exploratory access or when compliance forbids copies. Copy and model it when it's queried often, needs joins at scale, or must be historised and tested, because federated queries depend on the source's performance and availability.

**Your experience hook:** Building customer-facing RAG systems makes the AI cost of fragmentation concrete: answers are only as good as the documents the retriever can reach.

---

## Self-test

1. What are the four components of an EDM framework according to IBM, and how does MDM fit in?
<details><summary>Answer</summary>Data governance, metadata management, technical enablers (integration, ETL, MDM, profiling, lineage tools) and data lifecycle management. MDM is a subset of EDM focused on keeping core entities consistent; it depends on EDM's governance, ownership and standards.</details>

2. What is the difference between a CIO and a CDO?
<details><summary>Answer</summary>The CIO is responsible for the technology infrastructure that processes data; the CDO is responsible for how data is used to create business value (strategy, governance, quality, analytics).</details>

3. Name MIT's four core data literacy abilities.
<details><summary>Answer</summary>Read data, work with data, analyse data and argue with data.</details>

4. What is a data swamp and how does democratization try to avoid one?
<details><summary>Answer</summary>A data lake with no quality or governance practices, so its data becomes unusable. Democratization counters it with governance, oversight and treating data as a product with owners.</details>

5. List the six parts of IBM's data SLA template.
<details><summary>Answer</summary>Purpose, promise, measurement, ramifications, requirements (what you need from consumers) and signatures.</details>

6. Explain SLI, SLO and SLA using a freshness example.
<details><summary>Answer</summary>SLI: the measured value, e.g. hours since the table was last loaded. SLO: the internal target, e.g. under 2 hours on 99% of days. SLA: the agreement with consumers (e.g. data ready by 05:00) including what happens when it is missed.</details>

7. Give four common causes of data silos.
<details><summary>Answer</summary>Any four of: organisational structure (team-specific tools), IT complexity and legacy systems, a culture of treating data as departmental property, resource constraints, regulatory requirements such as regional separation, and mergers or rapid growth.</details>

8. What is the difference between dimensionality reduction and numerosity reduction?
<details><summary>Answer</summary>Dimensionality reduction cuts the number of attributes/features (e.g. PCA). Numerosity reduction replaces the data values with a smaller representation, using parametric models (regression) or non-parametric summaries (histograms, sampling).</details>

9. Does solving data fragmentation mean putting all data into one store? Why or why not?
<details><summary>Answer</summary>No. Hybrid multicloud estates, volume and compliance make that unrealistic. The aim is to connect the right data to the right people at the right time, using consolidation where it pays off plus integration, federation, data fabric and governance.</details>

10. What are the six components of a data strategy in IBM's article?
<details><summary>Answer</summary>Understanding business objectives, assessing obstacles and challenges, creating a roadmap, establishing controls, focusing on small wins and scaling the strategy.</details>
