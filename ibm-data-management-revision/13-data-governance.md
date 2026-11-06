# 13. Data governance

> This short section covers the rules and the people behind trustworthy data: data governance (policies, standards, roles, access and auditing) and data stewardship (the day-to-day work that puts those policies into practice). In interviews, governance comes up in fintech and regulated settings ("how do you handle PII?", "who owns this table?") and in lead roles ("how would you introduce governance without slowing everyone down?"). Good answers are concrete: named owners, access by role, classification and tags, lineage, audit logs and automated tests, not just policy documents.

## Data governance
*Source: [IBM — What is Data Governance?](https://www.ibm.com/think/topics/data-governance)*

**In one line:** Data governance is the data management discipline that keeps data high-quality, secure and available by defining and enforcing policies, standards and procedures for how data is collected, owned, stored, processed and used.

**Key points**
- IBM compares it to air traffic control: verified data flows through secured pipelines to trusted users. AI, big data and digital transformation drive it, and it now also has to cover RAG inputs and outputs, vector databases and AI agents that access data.
- A **framework** is tailored to each organisation and covers: goals and metrics (fewer errors and duplicates, cost savings, consistency and completeness, literacy and policy compliance); **roles**; standards, policies and processes (formats, models, MDM, metadata, naming, data flows, **data scopes** for access); auditing procedures; and tools.
- Roles: **steering committee or governance council** (executives setting direction), **data owners** (accountable for a domain's accuracy and quality), **data stewards** (day-to-day management of a domain) and **stakeholders** (the data consumers).
- Tool capabilities: automatic discovery and classification, enforcement of protection rules and role-based access control, privacy and compliance support, automated metadata, cataloguing and lineage, and a business glossary.
- Benefits: more value from data, innovation (appropriate access instead of reliance on a few experts), a single source of truth through a central catalog, compliance (GDPR, HIPAA, PCI DSS, EU AI Act, EU Data Act, DORA) and safe use of data for AI (e.g. keeping PII out of models).
- Challenges: missing sponsorship (needs both a CDO and stewards), inconsistent architecture, poor visibility across hybrid multicloud and shadow IT, rising self-service access demand, and AI's extra requirements.
- Best practices: automate (lineage, policy-driven sensitivity tags on columns, audit logs, classification into PII, financial, IP and confidential), balance convenience with safety, build a data catalog, use a maturity model, and monitor and improve continuously.

**Compare / don't confuse with:** *Data governance vs. data management* — governance is a subset of data management. Governance sets the policy (e.g. who may see PII); data management implements it (e.g. building role-based access, the data model, the integration).

**Interview angle**
- *Q: How would you introduce data governance at a small company without slowing teams down?* — Start with the highest-risk and most-used data: classify PII, name an owner per key dataset, put access on roles and groups instead of individuals, and document and test the core models. Automate as much as possible (tags, masking policies, lineage from the transformation tool) so governance is built into the platform rather than handled through manual approvals.
- *Q: How do you control access to PII in an analytics warehouse?* — Classify the columns, apply column-level security or masking (e.g. policy tags), use row-level policies where tenants or regions must be separated, give analysts pseudonymised keys instead of raw identifiers, and log access for audits.

**Your experience hook:** PII pseudonymisation and access control on the multi-tenant KYC/KYB platform are governance policies implemented in code, the "data management implements governance" split in practice.

**Beyond IBM:** Modern practice favours "governance as code": access policies, tags and tests live in version control and are applied through CI/CD, and data contracts between producers and consumers make schema and quality expectations explicit and enforceable.

---

## Data stewardship
*Source: [IBM — What Is Data Stewardship?](https://www.ibm.com/think/topics/data-stewardship)*

**In one line:** Data stewardship is the hands-on set of practices, run by data stewards, that keeps data high-quality and accessible in line with the organisation's governance policies.

**Key points**
- Steward tasks include defining quality metrics, managing metadata and reference data, tracing lineage and classifying sensitive data. With AI, stewardship also helps keep AI inputs and outputs compliant and ethical.
- Benefits: better quality, access, usability and security; consistent interpretation of data; easier audits. Many people do stewardship work without the title, but formalising the role signals a company takes quality seriously.
- Types of steward: **business** (one function, e.g. marketing), **technical** (ETL and warehouse expertise) and **enterprise** (leads the steward community and links to business leaders).
- Use cases: MDM (often starting with one domain such as customer), quality improvement (profiling, definitions, valid-value rules, e.g. allowed marital-status values), metadata management, **reference data** (country, currency and product codes, reconciled across systems), **identity resolution** (recognising that records refer to the same entity), security classifications, lineage for regulatory reporting, and business-process risk assessment.
- Supporting tools: AI (validation, sensitive-data detection), data catalogs, profiling tools, relational databases and governance software (metadata enrichment, lineage, role-based access).

**Compare / don't confuse with:** *Stewardship vs. governance* — governance defines the policies, standards and accountability; stewardship is "the operational aspect" that carries them out day to day. *Steward vs. owner* — the owner is accountable for a data domain; the steward does the daily work of keeping it in order.

**Interview angle**
- *Q: What is reference data and why does it need a steward?* — Reference data is the small set of codes used to categorise other data (country codes, currencies, status values). Different systems often use different code lists, so someone has to own the canonical list, its valid values and the mappings between systems; otherwise joins and reports quietly disagree.

**Your experience hook:** Entity resolution with Splink is identity resolution as the article describes it: deciding when different records represent the same customer, with blocking to keep the comparison tractable.

**Beyond IBM:** In small teams stewardship is a hat, not a job title: the engineer or analyst who owns a dataset maintains its documentation, tests and accepted values. Making that ownership explicit (e.g. an `owner` field in model metadata) is the lightweight version.

---

## Self-test

1. Define data governance and explain how it differs from data management.
<details><summary>Answer</summary>Governance is the discipline of setting and enforcing policies, standards and procedures so data is high-quality, secure and available. It is a subset of data management: governance decides the rules (e.g. who may see PII), while data management builds and runs the systems that implement them (access control, models, integration).</details>

2. Name the four roles in a typical governance framework and what each does.
<details><summary>Answer</summary>Steering committee or governance council (sets strategy and direction), data owners (accountable for quality and accuracy of a domain), data stewards (daily management of a domain) and stakeholders/business teams (data consumers).</details>

3. What is a data scope?
<details><summary>Answer</summary>An access boundary for specific data assets (e.g. master data, metadata, historical data) so users and apps reach only the data they need.</details>

4. List five common capabilities of data governance tools.
<details><summary>Answer</summary>Automatic discovery and classification, enforcement of protection rules and role-based access control, privacy and compliance support, automated metadata management, cataloguing and lineage, and a business glossary.</details>

5. What are the main challenges of data governance according to IBM?
<details><summary>Answer</summary>Lack of sponsorship (executive and individual-contributor level), inconsistent data architecture, poor visibility and control across hybrid multicloud environments and shadow IT, growing demand for self-service access, and the extra demands of AI data.</details>

6. How does data stewardship relate to data governance?
<details><summary>Answer</summary>Stewardship is the operational side of governance: stewards apply the governance policies day to day by defining quality rules, managing metadata and reference data, tracing lineage and classifying sensitive data.</details>

7. Name the three types of data steward.
<details><summary>Answer</summary>Business data steward (a specific business function), technical data steward (ETL, warehouses, systems) and enterprise data steward (leads the steward community and liaises with business leaders).</details>

8. What is identity resolution, and give an example.
<details><summary>Answer</summary>Determining when different records refer to the same real-world entity. For example, a pharmacy customer who appears several times because prescriptions were filled at different stores; resolving them into one identity lets the pharmacy detect dangerous drug interactions.</details>
