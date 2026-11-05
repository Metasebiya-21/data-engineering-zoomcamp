# Apekso — Interview Prep (built from the job post and your resume)

## 0. What they're testing, and where your resume already answers it

The job post: *"We build and run data-intensive software: backend services, databases, web interfaces, and the infrastructure under them."* They shortlisted you for your **data engineering** background. Lead with it.

| They ask for | Your evidence (from your resume) |
|---|---|
| Software real users depended on in production, 3+ years | MICHU lending platform: **6 banks, 2M+ users, 30B+ ETB disbursed**. KYC/KYB platform with **2M+ records**. 4+ years. |
| Design database schemas and data flows | Consolidated KYC/KYB data from multiple source databases. UUID surrogate keys to fix ID and phone-number collisions. MongoDB → Oracle normalisation. Dimensional modelling. |
| SQL, Linux, working problems out from logs | Expert SQL. PostgreSQL tuned for **~900 req/s**. Traced incidents through logs and metrics to root cause. Prometheus, Grafana, ELK. |
| Find your way around code you didn't write, decide, explain | Moved from software engineering into lending data. Led 5 engineers with code review practices. Introduced hexagonal architecture. |
| Uses AI coding tools and rigorously tests and reviews the output | **n8n workflows orchestrating Claude Code agents**. Automated testing and data validation. |
| Docker / Kubernetes, monitoring, fixing failures | Docker, CI/CD, Azure, Prometheus/Grafana/ELK. Added monitoring after the CDC stall. |
| Backend services and integrations | FastAPI, Django and Node.js services. Webhooks and API integrations. RabbitMQ and Redis queues. |

**Your one-line positioning:** *"I build and run data systems that people depend on, and I'm at my best when something breaks and I have to work out why from the logs and the data."*

Their hiring process has 3 steps. Prepare for each one:
1. **Describe a software or data system you built and ran** (§1)
2. **A practical task using your normal AI tools.** They watch how you direct the tools, check the output, and explain the code (§2)
3. **A conversation about the work and how you'd approach it** (§3–§6)

---

## 1. Step 1: "Describe a system you built and ran"

### Main story: the KYC/KYB consolidation platform (Kifiya)
It's your strongest fit because it covers schema design, data flow, a production failure you diagnosed, reliability patterns, PII, and team leadership.

**Structure (about 5 minutes; fill in the blanks with real details, don't improvise numbers):**

1. **The problem (30 s).** "KYC/KYB records for [which products or tenants] lived in [N] source databases, [which ones: Postgres? MongoDB?]. We needed one consistent, multi-tenant view for onboarding and verification, serving [who: banks, internal ops]."
2. **What you built (90 s).** Draw the flow: *sources → CDC → [Redpanda/RabbitMQ] → consumers → consolidated store → services/API*.
   - Hexagonal architecture: core logic separated from adapters (databases, queues), so sources could be swapped and tested.
   - **UUID surrogate keys:** source IDs collided across databases, and phone numbers were reused or duplicated. Explain *why* you didn't use natural keys.
   - **Reliability:** idempotent consumers, retries with backoff, dead-letter queues.
   - **PII pseudonymisation and access control.**
3. **What broke and how you found it (90 s). This is the part they care about most.** "CDC streams **stalled silently**: no errors, the data just stopped arriving." How did you notice (a user complaint? a row-count mismatch? stale timestamps)? What did the logs show? What was the root cause?
   - The fix: **resume tokens** (a restarted consumer continues from its last processed position instead of starting over or skipping) plus **monitoring** (alert when the time since the last event or the consumer lag passes a threshold).
4. **The result (30 s).** A number if you have one: records consolidated, incidents after the fix, detection time going from days to minutes.
5. **What you'd do differently (30 s).** Shows judgment. For example: "I'd have built freshness and lag alerts on day one instead of after the incident."

**Follow-up questions to rehearse:**
- *Which CDC mechanism?* MongoDB change streams, Debezium on Postgres, or something else? **What exactly is a resume token, and where did you store it?** If the token is older than the oplog/WAL retention window, you can't resume and you need a re-snapshot. What was your plan for that?
- *Why did it stall silently?* Typical causes: a consumer stuck but not crashed, a cursor or connection that timed out without raising an error, an expired resume position, or a filter that dropped everything. Which was yours?
- *How is a consumer idempotent?* What is the idempotency key (event ID, or source key plus version)? Upsert vs. "insert if not exists"? What about ordering (an update arriving before its insert)?
- *What happens to messages in the DLQ?* Who looks at them? How do you replay them?
- *How did you pseudonymise?* Keyed hashing (HMAC) vs. a tokenisation table vs. encryption. Who can re-identify a person, and how is that access controlled and audited?
- *How did you deduplicate people across sources?* This links to your entity resolution project below.
- *Multi-tenancy:* how was tenant data kept separate (tenant_id on every row, separate schemas, row-level security)?

### Backup story A: MongoDB → Oracle ETL (Airflow)
Good if they want something more *data-flow* focused: nested documents → normalised relational schema, change-stream capture **including deletes**, plus **initial-load, incremental and full-refresh modes**.
- Be ready for: how you flattened arrays (child tables with foreign keys), how deletes were applied, how the run is made idempotent (re-runs after a failure), and how an initial load hands off to incremental sync without gaps or duplicates (record the stream position *before* the snapshot starts).

### Backup story B: MICHU, PostgreSQL at ~900 req/s
Good for "databases under real load." Know *what you tuned*: which queries (found with `pg_stat_statements` or slow query logs?), indexes added, `EXPLAIN ANALYZE` findings, connection pooling, N+1 queries, caching. Have one before/after number.

### Backup story C: Entity resolution (Splink)
"Blocking reduced 14.7B candidate pairs to 77M." Be ready to explain blocking: only compare records that share a cheap key (e.g. same phone prefix, or same date of birth plus first initial). Explain precision vs. recall, and how you checked match quality.

---

## 2. Step 2: practical task with your normal AI tools

**What they say they're judging:** *how you direct the tools, how you check the output, and whether you can explain the code afterwards.* The result alone isn't enough. They're watching your **process**.

For a data-focused candidate, likely tasks: build a small ingestion or integration (an API → database), design a schema plus queries for a scenario, fix a failing pipeline or service from its logs, or add a feature with tests to existing code.

### How to work (say this out loud as you go)
1. **Read the task and the existing code before prompting.** Restate the task in your own words. Ask clarifying questions (data volume, duplicates, what "done" means).
2. **Give the AI the context and constraints:** the schema, an example of the input, the edge cases, and "don't change X."
3. **Ask for small steps, not the whole thing at once.** Review each diff before moving on.
4. **Write or ask for tests for the edge cases first:** empty input, duplicates, NULLs, malformed rows, re-runs.
5. **Run it.** Check the output with your own SQL: row counts, sums, duplicate check, spot-check a few records against the source.
6. **Reject or rewrite anything you can't explain.** Say so: "I'm not keeping this part until I understand it."

### Review checklist for AI-generated data code (your differentiator)
- **Errors swallowed** (`except: pass`, ignored return values) → silent data loss. *Your CDC story is exactly this failure.*
- **SQL built with string formatting** → injection. Use parameters.
- **Not idempotent:** a re-run inserts duplicates. Use upsert, `ON CONFLICT`, or delete+insert per partition.
- **No unique constraint:** dedup in the application but not enforced in the database.
- **Missing transaction:** a partial write on failure.
- **Money as float** (a fintech habit to show!) → use `NUMERIC`/`Decimal`.
- **Time zones:** naive timestamps. Use `timestamptz`, store in UTC.
- **NULL handling:** `NOT IN` with NULLs, `COUNT(col)` vs. `COUNT(*)`, joins that silently drop rows.
- **No `ORDER BY`** but the code assumes an order. Pagination with `OFFSET` that skips or repeats rows while data changes (use keyset pagination).
- **Retries with no backoff, or retrying non-transient errors.** No timeouts on HTTP or database calls.
- **Loading everything into memory** instead of chunking or streaming.
- **Comments or docstrings that claim behaviour the code doesn't have.** AI often does this.
- **Hallucinated library functions or parameters.** Run it, don't trust it.

### Afterwards: explaining the code
Walk through it as **data flow → key decisions → failure handling → how you tested it → what you'd add with more time** (monitoring, alerts, documentation).

---

## 3. Schema design and data flows (in their list of what you'll do)

Practise one out loud: **"Design the schema for a multi-tenant loan application and verification system"** (it plays to your experience).
- Entities: `tenants`, `customers`, `applications`, `documents`, `verifications`, `events/audit_log`.
- **Keys:** UUID surrogate primary keys (your collision story), natural keys as `UNIQUE` constraints per tenant, e.g. `UNIQUE (tenant_id, national_id)`.
- **Constraints do the work:** `NOT NULL`, `CHECK (amount > 0)`, foreign keys, enums or lookup tables for status.
- **Status changes:** current status on the row plus an append-only history or event table (who, when, from → to). This makes audits easy.
- **Types:** `NUMERIC(18,2)` for money, `timestamptz`, `JSONB` only for genuinely variable payloads.
- **Indexes from query patterns:** `(tenant_id, status, created_at)`. A partial index for "pending" items.
- **Tenant isolation:** `tenant_id` on every table, plus Postgres row-level security.
- **PII:** separate table or column-level encryption, pseudonymised IDs in analytics copies.
- **Schema changes on a live system:** expand → migrate → contract (add the new column, backfill, switch reads, then drop the old one), so there's no downtime.
- **Data flow out:** OLTP → CDC → warehouse for analytics, never heavy queries on the primary.

---

## 4. SQL, Linux and logs

**SQL tasks to be fluent in:** window functions (`row_number` for dedup, `lag` for changes, running totals), latest record per key, gaps in sequences, a funnel per status, `EXPLAIN ANALYZE` reading. Drills are in `INTERVIEW_PREP.md` §4.

**Debugging from logs: a method to say out loud:**
1. Define the symptom precisely: what, since when, for whom.
2. Check what changed: deploys, config, upstream data, volume.
3. Find one failing request or record ID and follow it through the logs (correlation or request IDs).
4. Form a hypothesis → check it → fix → **add the alert or test that would have caught it**.

**Linux commands to know:** `tail -f`, `less +F`, `grep -i -C 3`, `grep -c`, `awk '{print $9}' | sort | uniq -c | sort -rn` (count by field), `jq` for JSON logs, `journalctl -u svc --since "1 hour ago"`, `docker logs -f --since 10m`, `df -h` / `du -sh` (full disk is a classic cause), `free -m`, `top`/`htop`, `ss -tlnp` (what's listening), `curl -v`.

---

## 5. Gaps in the job post: answer honestly and briefly

You didn't list Go, Kubernetes or TypeScript, and they shortlisted you anyway. If it comes up, don't over-explain:

> "I haven't used Go in production. My backend work has been Python and Node.js. The concepts carry over (HTTP services, queues, database access, testing), and I'd expect to get productive in Go quickly, with AI tools helping me learn while I review everything carefully. My strength is the data side: schemas, data flows, and keeping pipelines correct in production."

Kubernetes: know the vocabulary for a conversation (pod, deployment, service, ConfigMap/Secret, readiness and liveness probes, `kubectl logs` / `describe`, CrashLoopBackOff, OOMKilled). Say you've worked with Docker and CI/CD on Azure.

---

## 6. Questions about your resume to expect
- **"You started at EagleLion in June 2026. Why look now?"** Have a short, positive answer about what you want next (e.g. building and running software used directly by users, a broader engineering role). Don't criticise the current employer.
- **"Architect, Data Engineer / Scientist": what's your actual day-to-day?**
- **"How do you check what your n8n Claude Code agents produce?"** This maps *directly* to their AI criterion. Be ready with specifics: tests, human approval steps, logging, limits on what agents can do, examples of output you rejected.
- **"Tell me about leading 5 engineers."** Code review standards, testing, how you handled disagreement.
- **"PCA for credit scoring: how did you keep it interpretable?"** Factor loadings mapped to business meaning. Be ready for trade-offs vs. simpler scorecards.

---

## 7. Questions to ask them
1. What kind of data-intensive systems are you building right now, and for which clients or users?
2. What does the data side look like: databases, volumes, pipelines?
3. How does the team use AI coding tools day to day, and how do you review what they produce?
4. When something breaks in production, how is it found and who handles it?
5. How much time goes to working directly with users, as the job post mentions?
6. What would a successful first 3 months look like in this role?

---

## 8. Cheat sheet for the morning
- Lead with **production + data + debugging**: MICHU (6 banks, 2M+ users), KYC/KYB (2M+ records), 900 req/s Postgres.
- **System story:** problem → architecture → *silent CDC stall* → resume tokens + monitoring → result → what you'd change.
- **AI task:** clarify → give context → small steps → tests first → run and check with SQL → reject what you can't explain.
- **Reliability words:** idempotency key, upsert, unique constraint, retry with backoff, DLQ, resume position, lag/freshness alert.
- **Schema words:** surrogate vs. natural keys, constraints, `NUMERIC` for money, `timestamptz`, expand/contract migrations, row-level security.
- **Gaps:** honest, one sentence, then back to your strengths.
