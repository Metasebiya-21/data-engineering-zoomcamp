# 5. Data architecture

> This section moves up from individual storage systems to the blueprint that connects them: data architecture as a whole, the two big organising patterns (data fabric and data mesh), and the data modelling that gives every system its structure. Lead and senior interviews lean on this material. Expect "centralised or decentralised, and why?", "fabric vs. mesh", "walk me through conceptual → logical → physical models", and "how would you model this domain?". Strong answers tie each pattern to team size, organisational maturity and concrete trade-offs rather than buzzwords.

## Data architecture
*Source: [IBM — What Is a Data Architecture?](https://www.ibm.com/think/topics/data-architecture)*

**In one line:** Data architecture is the blueprint for how data is collected, moved, stored, governed and consumed across an organisation.

**Key points**
- Vocabulary: **platform** (the technology that runs the systems), **data model** (entities, relationships, formats), **framework** (a design methodology), **component** (a pipeline, lake, catalog and so on) and **pattern** (a reusable solution such as fabric or mesh).
- Two broad approaches: **centralised** (one lake or warehouse under one governance model) and **decentralised** (domains own their data, often event-driven or NoSQL). Most organisations run a hybrid.
- Data models come at three levels: conceptual, logical and physical (see Data modeling below).
- Frameworks: **TOGAF** (business, data, application and technology architectures), **DAMA-DMBOK 2** (the data management body of knowledge) and **Zachman** (a 6×6 grid of perspectives against what/how/where/who/when/why; it classifies but gives no method).
- Components fall into four groups: flow and integration (pipelines, APIs/connectors), storage (warehouse, mart, lake, lakehouse, databases), access and consumption (BI, query engines, embedded data products, ML training) and governance/metadata (lineage, observability, catalogs).
- Implementation: business needs → models and governance policies → technology choices → deploy pipelines, APIs and access points with security built in → monitor and refine continuously.
- Modern traits: cloud-native, streaming-capable, API-first, decoupled services, domain-oriented, cost-aware. Benefits: less redundancy, better quality, integration, and lifecycle tiering (moving old data to cheaper storage).

**Compare / don't confuse with:** Data architecture (how data flows between systems) vs. data modelling (how data is structured inside a system). Also data architecture vs. data platform: the platform is the concrete tooling that implements the architecture.

**Interview angle**
- *Q: You join a small company with data spread across app databases and spreadsheets. What architecture do you propose?* — Start centralised and simple: CDC or batch extracts into one warehouse, layered models (staging → core → marts), an orchestrator, basic quality checks, lineage and access control. Decentralise only when the number of teams makes the central team a bottleneck.

**Your experience hook:** Leading five engineers on the multi-tenant KYC/KYB platform meant choosing how data from several source databases flowed through CDC into one consolidated store, with access control and PII pseudonymisation designed in rather than added later.

**Beyond IBM:** Other patterns you will be asked about are **Lambda** (separate batch and speed layers, merged at query time) and **Kappa** (a single streaming path, reprocessing by replaying the log). Kappa is attractive with Kafka/Redpanda retention, but replays must be idempotent.

---

## Data fabric
*Source: [IBM — What Is a Data Fabric?](https://www.ibm.com/think/topics/data-fabric)*

**In one line:** A data fabric is a design approach that uses metadata, automation and virtualisation to give unified, governed, self-service access to data wherever it physically lives.

**Key points**
- It is a design, not a product. It connects on-premises and multicloud sources (lakes, warehouses, databases) without forcing everything into one store, and also without full decentralisation.
- Motivation: silos and **dark data** (data collected but never used) from the spread of platforms; IBM cites estimates that around 60% of enterprise data goes unanalysed. It also aims to give AI governed, trusted inputs.
- Core capabilities: catalogs driven by **active metadata** (knowledge graphs, semantics, ML), integration in many styles (batch, real-time, CDC), centrally defined governance applied automatically through metadata tags, a self-service data marketplace, and a unified lifecycle for pipelines.
- The three building blocks behind it: **data virtualisation** (query sources in place through a virtual layer instead of copying them with ETL), **federated active metadata** (continuous automatic profiling, tagging and classification that can trigger actions) and **ML automation** (policy enforcement, lineage, quality fixes).
- Forrester's six layers: management, ingestion, processing, orchestration, discovery and access.
- Benefits: efficiency, data democratisation, reduced risk (masking and encryption driven by classification), and horizontal and vertical scalability. It is meant to counter **data gravity**, the growing difficulty of moving data as volumes increase.

**Compare / don't confuse with:** Fabric vs. mesh: a fabric is a technology-led integration and governance layer, usually run centrally; a mesh is an organisational model of domain ownership. IBM presents them as complementary: a fabric can automate data-product creation and global governance for a mesh. Fabric vs. lakehouse: a lakehouse is a storage architecture; a fabric is an access and governance layer that can sit over a lakehouse and other stores.

**Interview angle**
- *Q: When would data virtualisation be a bad choice?* — For heavy, repeated analytical queries or joins across large remote sources. Every query hits the operational systems, latency depends on the slowest source, and there is no history. Materialise (replicate or CDC into a warehouse) when performance, source load or point-in-time history matter.

**Beyond IBM:** "Data fabric" is largely an analyst and vendor term. IBM calling it "the next stage" after the lakehouse reflects its own product positioning. In practice it means a good catalog with automated classification, a federated query engine such as Trino, and policy-based access control. Judge any fabric by whether its metadata is actually kept accurate.

---

## Data mesh
*Source: [IBM — What Is a Data Mesh?](https://www.ibm.com/think/topics/data-mesh)*

**In one line:** A data mesh is a decentralised, domain-oriented approach in which the teams that produce data own it and publish it as products for the rest of the organisation.

**Key points**
- Zhamak Dehghani (ThoughtWorks) proposed it to remove the bottleneck of a central data team and monolithic platform. It is often compared to microservices.
- Four principles: **domain-oriented decentralised ownership**, **data as a product**, **self-serve data infrastructure as a platform** and **federated computational governance**.
- A **data product** is a reusable, self-contained package of data, metadata, semantics and access interface, built with product thinking. It should be discoverable, understandable, interoperable, shareable, secure and reusable.
- Domains run their own ETL/ELT. The central data team does not disappear: it becomes the platform team (storage, orchestration, lineage, templates). Dehghani describes three planes: infrastructure provisioning, the data product developer experience, and mesh supervision.
- Governance is federated: domains set local quality, documentation and access rules, while a central group defines global standards (shared field types, IDs, metadata, security).
- Lakes and warehouses remain, but they support many domain-owned stores instead of being the single central platform.
- Claimed benefits: democratisation, cost visibility, elastic scaling, less central technical debt, interoperability through shared standards, and domain-level compliance. Use cases: BI, chatbots/RAG, customer 360, faster ML.

**Compare / don't confuse with:** Mesh vs. fabric (see the Data fabric entry). Also a mesh vs. simply "having many team databases": without product contracts, discoverability and global standards, decentralisation just recreates silos.

**Interview angle**
- *Q: Would you introduce a data mesh at a 30-person company?* — No. A mesh solves an organisational scaling problem (many domains overloading one central team), and it needs mature domain teams and a strong self-serve platform. A small company gets more from one well-run warehouse, with clear model ownership and data contracts at the source boundaries.

**Beyond IBM:** The practical entry point is **data contracts**: an agreed schema, semantics, SLAs and change policy between producer and consumer, checked in CI. Mesh projects commonly fail on the platform and governance side, not the technology side.

---

## Data modeling
*Source: [IBM — What Is Data Modeling?](https://www.ibm.com/think/topics/data-modeling)*

**In one line:** Data modelling is designing and documenting what data a system holds, how it is structured and how its parts relate, moving from business concepts down to a physical schema.

**Key points**
- Driven by business requirements gathered from stakeholders. Models should be living documents that change with the business.
- Three levels: **conceptual** (domain model: entities, relationships, business rules, simple notation, made early), **logical** (attributes, types and relationships in a formal notation, technology-agnostic; often skipped in agile but valuable for warehouse design) and **physical** (tables, primary and foreign keys, associative tables, indexes, DBMS-specific tuning).
- Process: identify entities → identify their attributes → identify relationships → map attributes fully → assign keys and choose a degree of normalisation → validate and iterate.
- Normalisation removes repetition by linking records through keys. It saves storage and protects integrity, but can cost query performance.
- Model families: **hierarchical** (trees; IBM IMS 1966, still seen in XML), **relational** (Codd 1970, SQL, the OLTP default), **entity-relationship** (diagram-based design), **object-oriented** (classes and complex relationships) and **dimensional** (Kimball; facts and dimensions, star and snowflake, deliberately redundant for fast analytics).
- Benefits: fewer development errors, consistent documentation, better performance, easier mapping, and clearer communication between developers and BI teams. Tools include erwin, ER/Studio, Enterprise Architect and Open ModelSphere.

**Compare / don't confuse with:** Relational/ER modelling (normalised, write-optimised, integrity-first, for OLTP) vs. dimensional modelling (denormalised, read-optimised, for OLAP). Also the logical model (what attributes and relationships exist) vs. the physical model (how they are stored on a specific engine).

**Interview angle**
- *Q: How do you handle a customer's address changing in a dimensional model?* — Use a Slowly Changing Dimension. Type 1 overwrites (no history). Type 2 adds a new row with a new surrogate key plus valid-from/valid-to dates and a current flag, so facts stay joined to the address that was true when they happened.
- *Q: Natural keys or surrogate keys?* — Surrogate keys for warehouse dimensions and integrated entities. Natural keys from different sources collide, get reused or change, while a surrogate stays stable; keep the natural keys as attributes for lookup and lineage.

**Your experience hook:** Merging records from several source databases, I replaced colliding source IDs and phone numbers as identifiers with UUID surrogate keys, a physical-model decision that made the consolidated store consistent.

**Beyond IBM:** IBM says relationships are documented in UML, but data models more commonly use ER notation (crow's foot, IDEF1X). For analytics, know the main options: Kimball star schemas, Inmon 3NF EDW, Data Vault 2.0 (hubs, links, satellites; auditable and good for many changing sources) and wide one-big-table designs that work well on columnar engines.

---

## Self-test

1. Name IBM's five key terms in data architecture and give an example of each.
<details><summary>Answer</summary>Platform (e.g. a cloud environment), data model (entities and relationships), framework (TOGAF), component (a pipeline, lake or catalog) and pattern (data fabric or data mesh).</details>

2. Contrast centralised and decentralised data architectures, and say which most companies end up with.
<details><summary>Answer</summary>Centralised: data in one lake or warehouse under one governance model, giving less redundancy and consistent quality. Decentralised: domains own and manage their data, giving agility and real-time, domain-specific processing. Most organisations run a hybrid.</details>

3. What are the four pillars of TOGAF, and what does Zachman provide that it doesn't?
<details><summary>Answer</summary>Business, data, application and technology architecture. Zachman is a classification grid (perspectives × what/how/where/who/when/why) for organising architecture artefacts. It provides no method; TOGAF provides a process.</details>

4. What are the three building blocks of a data fabric?
<details><summary>Answer</summary>Data virtualisation (query data in place without copying it), federated active metadata (continuous automated profiling, tagging and classification) and machine learning (automating governance, lineage and quality).</details>

5. Define "data gravity" and "dark data".
<details><summary>Answer</summary>Data gravity: as data grows, it becomes harder and costlier to move, so applications and services move towards it. Dark data: data collected and stored but never analysed or used.</details>

6. List Dehghani's four data mesh principles.
<details><summary>Answer</summary>Domain-oriented decentralised ownership; data as a product; self-serve data infrastructure as a platform; federated computational governance.</details>

7. In a data mesh, what happens to the central data engineering team?
<details><summary>Answer</summary>It becomes a platform team. It provides the self-serve infrastructure (storage, orchestration, lineage, templates) and global standards, while domain teams build and own their data products and pipelines.</details>

8. Is fabric vs. mesh an either/or choice? Explain.
<details><summary>Answer</summary>No. A mesh is an organisational ownership model; a fabric is a metadata-driven integration and governance layer. A fabric can automate a mesh's data-product creation and global policy enforcement.</details>

9. Describe the conceptual, logical and physical data models and who each is for.
<details><summary>Answer</summary>Conceptual: entities, relationships and business rules, for business stakeholders. Logical: attributes, types and relationships in a formal, technology-agnostic notation, for analysts and architects. Physical: tables, keys, indexes and engine-specific details, for engineers and DBAs.</details>

10. Why do dimensional models deliberately add redundancy, and how do you keep history of changing dimension attributes?
<details><summary>Answer</summary>Denormalised dimensions mean fewer joins and simpler, faster analytical queries. History is kept with SCD Type 2: a new row with a new surrogate key, effective-date columns and a current flag for each change.</details>
