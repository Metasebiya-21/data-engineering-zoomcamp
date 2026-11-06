# My Production Experience, Mapped to Data Management Concepts

Two systems, each told as an interview story, with the vocabulary from the [IBM revision](ibm-data-management-revision/README.md) attached. Use this for "describe a system you built and ran" and for any "have you done X?" question.

---

## System 1: MICHU digital lending platform

### The facts
- Digital lending platform used by **6 banks**. **MICHU**, the pioneering lending platform, was deployed for **Coop Bank** and has disbursed **70 billion ETB** to date.
- Built an **ETL service using batch processing**: data fetched from source databases into destination databases for **business intelligence and predictive analysis**.
- Then **replaced the batch processing with event streaming using CDC** to capture data changes in real time.

> ⚠️ **Make the numbers consistent.** Your resume says "over 30 billion ETB". If 70B is the current figure, say: *"over 30 billion when I wrote my resume; about 70 billion today."* Interviewers notice mismatches.

### Concepts this story demonstrates
| Concept (chapter) | Where it shows up |
|---|---|
| OLTP vs. OLAP (03, 09) | Source databases serve the lending app; destination databases serve BI and ML. Analytics moved off production |
| ETL vs. ELT (08) | The original batch ETL service: transform before loading into the destination |
| Batch processing → stream processing (09) | The migration from scheduled batch to event streaming |
| **Change data capture** (08) | Log-based capture of inserts, updates and deletes |
| Data streaming platform (07) | Redpanda/Kafka as the transport for change events |
| Data latency (07) | The *reason* for the move: batch data was hours old, CDC makes it seconds old |
| Data pipeline, orchestration (06) | The batch ETL scheduling (Airflow) |
| Feature store / AI-ready data (04, 01) | Predictive analysis on the destination data (credit risk, default prediction) |

### The story (2–3 minutes)
1. **Context.** "MICHU is a digital lending platform serving 6 banks. The flagship deployment for Coop Bank has disbursed about 70 billion birr. The lending services ran on [Postgres / which databases?]."
2. **First version: batch ETL.** "For business intelligence and predictive analysis, I built an ETL service that pulled data from the source databases on a schedule [every N hours / nightly], transformed it and loaded it into destination databases. That kept heavy analytical queries off the production databases."
3. **The problem with batch.** Pick the real reason(s): data was [N hours] stale, so risk and collections teams worked from yesterday's numbers; full or incremental extracts put load on production during the run; `updated_at`-based extraction **missed deletes** and missed rows when timestamps weren't updated.
4. **The change: CDC event streaming.** "I replaced the batch extraction with log-based CDC. Changes are read from the database's transaction log [WAL / binlog / oplog], published as events to [Redpanda / Kafka], and consumers apply them to the destination. Latency went from [hours] to [seconds or minutes], and the load on the source dropped because we read the log instead of querying tables."
5. **What you had to get right.** Initial snapshot then streaming handoff (no gaps or duplicates), idempotent consumers, ordering per key, handling deletes, schema changes, monitoring lag.
6. **Result.** Fresher data for BI and models, less load on production, [any number you have].

### Follow-ups to prepare (fill in the real answers)
- *Which CDC tool and log?* Debezium on Postgres WAL, MySQL binlog, MongoDB change streams, or custom?
- *How did you do the initial load and switch to streaming without losing or duplicating data?* Record the log position *before* the snapshot, then stream from that position; duplicates are harmless because consumers upsert.
- *What happens when a consumer is down for a day?* It resumes from its stored offset or resume token, **as long as the log is still retained**. Know your retention, and what you do if it's exceeded (re-snapshot).
- *How do you handle deletes?* Delete events or tombstones → hard delete or soft-delete flag in the destination.
- *How do you know it's working?* Consumer lag, time since last event, row-count reconciliation between source and destination. (Your silent-stall incident is the perfect example; see System 2.)
- *Why not just run batch more often?* Every batch run still re-queries production and still misses deletes. CDC reads each change once, from the log.
- *Did you keep batch at all?* Many teams keep a periodic full reconciliation as a safety net. If you did, say so.

---

## System 2: Digital onboarding and verification platform (KYC/KYB)

### The facts
- A platform for **onboarding customers with KYC/KYB** (verifying individuals and businesses), serving multiple banks.
- You **designed the service architecture and the data architecture**, including:
  - **Data governance** and **security for data at rest and in transit**.
  - **Tenant isolation:** *one bank must never see another bank's customers.*
  - **Secure data exchange** between the onboarding platform and the lending platform.
- **To solve data mismatches:**
  - **Data standardisation** at KYC/KYB level, using **coded reference data** (region, woreda, business sector, line of business): match on codes instead of names.
  - **Record linkage** to identify the same customer across sources.
- **Disaster recovery:** backups every **12 hours**.
- **Single sign-on (SSO).**
- (From your resume) Led 5 engineers, hexagonal architecture, CDC consolidation of 2M+ records, resume tokens and monitoring after CDC streams stalled silently, UUID surrogate keys, idempotency, retries, DLQs, PII pseudonymisation.

### Concepts this story demonstrates
| Concept (chapter) | Where it shows up |
|---|---|
| Data architecture (05) | You designed it end to end |
| **Data governance, stewardship** (13) | Who may see what, policies, ownership |
| **Data security** at rest and in transit | Encryption, TLS, access control, PII pseudonymisation |
| **Multi-tenancy / access control** | One bank can't see another's customers |
| **Master data management** (04) | A single trusted customer record across banks and sources |
| **Reference data management** (part of MDM) | Coded region, woreda, sector and line-of-business lists |
| Data standardisation, cleaning (06) | Codes instead of free-text names |
| **Data integrity, consistency** (12) | Fixing mismatches, surrogate keys, constraints |
| Record linkage / entity resolution | Splink, blocking 14.7B → 77M candidate pairs |
| Data interoperability, data contract (08) | The agreed format for exchange with the lending platform |
| CDC + data observability (08, 12) | The silent CDC stall → resume tokens + monitoring |
| Disaster recovery, backup | 12-hour backups (RPO) |
| Identity and access management | SSO |

### Wording to tighten (interviewers will probe these)
1. **"Data fabric":** be careful. A data fabric is a specific architecture: a metadata-driven layer that connects and governs data *across* many systems without moving it. What you describe (tenant isolation, governance, secure exchange between two platforms) is **governance, multi-tenant access control and secure integration**. Say that. Only call it a data fabric if you really built a shared metadata/catalog layer over several sources. If asked, you can say: *"It had fabric-like elements: central governance policies applied across the platforms we integrated."*
2. **Backups aren't MDM.** MDM is your **reference data codes + record linkage + single customer record**. Disaster recovery is a separate topic. Keep them apart in your answer.
3. **12-hour backups = an RPO of up to 12 hours.** Expect: *"So you could lose 12 hours of KYC data?"* Have an answer:
   - What else limited data loss (database replication, WAL/oplog archiving, point-in-time recovery, CDC events still in Redpanda)?
   - What's your **RTO** (time to restore)? **Did you test restores?** An untested backup is a hope, not a plan.
   - If 12 hours was the only protection, say what you'd improve: continuous WAL archiving + PITR, a replica in another site, regular restore drills.

### The story (3–4 minutes)
1. **Context.** "Banks onboarding customers needed KYC for individuals and KYB for businesses. The data came from [N] source systems with inconsistent formats, and the same customer often appeared several times. I led a team of 5 and designed both the service and data architecture."
2. **Architecture.** Hexagonal services; CDC from the sources into the platform; idempotent processing with retries and DLQs.
3. **Tenant isolation.** "Rule one: a bank must never see another bank's customers." Say *how*: `tenant_id` on every row enforced by [row-level security / separate schemas / service-layer checks], tenant-scoped tokens from SSO, tests that try to read across tenants.
4. **Security.** At rest: [database or disk encryption], PII **pseudonymised** with [keyed hashing / tokenisation], so analytics can join without seeing raw identities. In transit: TLS everywhere, [mTLS / signed requests] for the exchange with the lending platform. **SSO** ([Keycloak / Azure AD / OIDC?]) for central identity and role-based access.
5. **The data mismatch problem.** "The same region or business sector was written many ways ('Addis Ababa', 'A.A', 'Addis'), so joining on names failed. I standardised on **coded reference data** (region, woreda, business sector, line of business): free text is mapped to a code once, then everything matches on codes. For people and businesses there's no clean code, so I used **record linkage** (Splink, with blocking to cut 14.7B candidate pairs to 77M) to find duplicates, and gave each resolved entity a **UUID surrogate key**."
6. **The incident.** "CDC streams stalled **silently**: no errors, data just stopped arriving. [How did you find out?] I added **resume tokens**, so consumers restart from their last position, and **monitoring** on lag and time-since-last-event, so a stall alerts us in minutes."
7. **Resilience.** "Backups every 12 hours, [plus replication/PITR if true], with [restore tests]."
8. **Result.** [2M+ KYC/KYB records consolidated; duplicates found; onboarding time; zero cross-tenant leaks; detection time after the monitoring].

### Follow-ups to prepare
- *Who owns the reference code lists, and how are new codes added?* Data stewardship (ch 13): a named owner, versioned lists, a change process.
- *What if a free-text value doesn't map to any code?* An "unmapped" queue for review, not a silent NULL.
- *How did you choose record-linkage thresholds? How do you handle false matches?* Precision vs. recall, a manual review band, the ability to unmerge.
- *What's the difference between your surrogate key and a national ID?* National IDs can be missing, reused, mistyped, and are PII. A surrogate key is stable and non-sensitive.
- *How is the KYC data shared with the lending platform?* API vs. events, which fields, consent, an agreed contract/schema, the audit log.
- *GDPR-style deletion or correction requests?* How you propagate them across the lending platform and any analytics copies.
- *How do you prove tenant isolation works?* Automated tests, audit logs, penetration testing, row-level security in the database rather than only in application code.

---

## One-line versions (for "tell me about yourself")
- **Lending:** *"I built the data pipelines for MICHU, a lending platform used by 6 banks that has disbursed about 70 billion birr, and moved it from batch ETL to CDC-based event streaming so BI and risk models work on near-real-time data."*
- **Onboarding:** *"I led the design of a multi-tenant KYC/KYB platform: strict isolation between banks, PII protection, coded reference data and record linkage as the master data layer, and CDC monitoring after we learned that pipelines can fail silently."*
