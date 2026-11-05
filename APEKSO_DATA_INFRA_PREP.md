# Apekso — Data Infrastructure Engineer: Interview Prep

## 0. What we know about Apekso (verify before the interview)

From public profiles (Crunchbase, Dealroom, Briter). These are thin and partly AI-generated, so **check them on apekso.com and LinkedIn tonight**:

- **Dubai-based, 1–10 people.** A small, senior-led team.
- **Product, "Flow":** an AI-native, **real-time** learning space where users **speak and draw together** on a shared canvas. As someone talks through a STEM concept, the system generates matching drawings.
- They also do **selected engineering work** (a studio model) in **real-time systems, infrastructure and applied AI**.
- Funding is unclear: one profile says no external funding, another lists a $3.3M pre-Series A (March 2026).

**What "Data Infrastructure Engineer" probably means there.** At a 10-person AI startup, it isn't "maintain the Airflow DAGs." Expect a generalist who owns:
1. **Event and data capture** from real-time sessions (voice, canvas strokes, AI actions).
2. **Storage and databases** (Postgres, object storage, maybe a vector DB).
3. **AI/LLM data plumbing**: transcripts, prompts and responses, evaluation datasets, RAG indexes, latency and cost tracking.
4. **Analytics**: product usage and learning outcomes.
5. **Platform basics**: cloud, IaC, CI/CD, observability, security and privacy (students, possibly minors).

**Your positioning in one line:** *"I build data infrastructure that is simple to operate for a small team, from real-time event capture to analytics and AI datasets, using managed services where they save time and owning the parts that matter for the product."*

---

## 1. Plan for tonight and tomorrow

| When | Do |
|---|---|
| Tonight, 30 min | Check §0 on apekso.com, LinkedIn and the job post. Note their stack: cloud, languages, anything about WebRTC/WebSockets, LLM providers. |
| Tonight, 60 min | §3 system design: draw "Flow's data platform" once on paper, timed to 35 min. |
| Tonight, 45 min | §2 fundamentals: say the answers out loud. Focus on Postgres and streaming. |
| Tonight, 15 min | Adapt your intro (§5) and pick 3 stories. |
| Tomorrow AM | Re-read §6 (cheat sheet) and §7 (questions to ask). No new material. |

---

## 2. Technical topics, ranked by likelihood

### 2.1 Real-time event pipelines (core to their product)
The zoomcamp material that maps here is `07-streaming/` (Kafka/Redpanda, Flink, windows, late events).

- **Capturing events from a real-time collaborative app.** Clients connect over WebSockets (or WebRTC for audio). The app server sends every event (`stroke_added`, `utterance_transcribed`, `ai_drawing_generated`, `session_joined`) to a log or queue. Each event gets an `event_id` (UUID), `session_id`, `user_id`, `event_type`, `client_ts`, `server_ts`, `schema_version` and a payload.
- **Why `client_ts` *and* `server_ts`?** Client clocks drift. Use the server time for ordering and the client time for UX latency analysis. This is the event-time vs. processing-time distinction from module 07.
- **Ordering:** guarantee it *per session*, not globally. Use `session_id` as the Kafka partition key, or one Redis Stream per session. Ordering per key matters for replaying a canvas.
- **Delivery semantics:** at-least-once plus idempotent writes (dedup on `event_id`). Exactly-once is rarely worth the complexity at their scale.
- **Choosing a queue for a small team:**
  - *Redis Streams:* simple, probably already in the stack for real-time presence, but limited retention and replay.
  - *Managed Kafka* (MSK, Confluent, Redpanda Cloud): durable replay and a large ecosystem, but costs more and adds operational load.
  - *Cloud-native* (Kinesis, Pub/Sub, SQS + Firehose → S3): the least operational work.
  - *Postgres-only to start* (outbox table + batch export): fine for the first thousands of users.
  - **Say:** "Start with the simplest option that gives durable replay. Move to Kafka when there are several consumers or throughput needs it."
- **Backpressure and burstiness:** a classroom of 30 students starts at 9:00 at the same moment. Buffer in the queue, scale consumers out, keep producers non-blocking, use a dead-letter queue for poison messages.
- **Hot vs. cold path:** the hot path (live canvas sync) must never be blocked by analytics writes. Analytics is asynchronous, sent fire-and-forget to the queue.

### 2.2 Databases: Postgres depth (very likely at a startup)
- **Indexes:** B-tree (the default), GIN (JSONB, full text), BRIN (huge append-only time series), partial indexes, composite index column order. Read `EXPLAIN ANALYZE`: seq scan vs. index scan, rows estimated vs. actual.
- **MVCC and VACUUM:** updates create new row versions, and VACUUM reclaims dead tuples. Autovacuum tuning, table bloat, transaction ID wraparound.
- **Partitioning:** declarative range partitioning by time for event tables. Dropping an old partition is cheap retention (much cheaper than `DELETE`).
- **Replication and HA:** streaming replication, read replicas (and replication lag), failover. Managed options: RDS/Aurora, Cloud SQL, Neon, Supabase.
- **Connection pooling:** PgBouncer, because WebSocket servers × connections can exhaust `max_connections`.
- **JSONB vs. columns:** JSONB for flexible event payloads, promoted to real columns once a field is queried often.
- **OLTP vs. OLAP:** don't run analytics on the production primary. Use a replica, then CDC or exports to a warehouse/lakehouse (BigQuery, ClickHouse, DuckDB on Parquet).
- **Backups and DR:** PITR (WAL archiving), *tested* restores, RPO/RTO targets.

### 2.3 Storage layout for session data
- **Canvas and session replay:** an event log (an append-only stroke and command stream) plus periodic snapshots (the full canvas state every N seconds). Replay = latest snapshot + the events after it. This is the same idea as Kafka log compaction and event sourcing.
- **Audio:** raw audio goes to object storage (S3/GCS) with lifecycle rules. Transcripts and metadata go in Postgres or the warehouse. Store large blobs by pointer, never inside the database.
- **The lake:** raw events land as **Parquet** in object storage, partitioned by `date=`/`event_type=`. Query them with DuckDB, Athena or BigQuery external tables (module 03). Add a table format (Iceberg/Delta) later if you need updates or deletes, e.g. for GDPR deletion.
- **Retention tiers:** hot (days) → warm (months) → cold archive. Set policies per data class, with audio kept for the shortest time.

### 2.4 AI/LLM data infrastructure (their product is AI-native)
- **Log every model call:** request ID, session, model, prompt version, input and output tokens, latency (time to first token and total), cost, errors. Prompts and responses go to storage (redacted where needed). Tools: LangSmith, Langfuse, Helicone, or your own table.
- **Evaluation datasets:** sample real sessions → label them (human or LLM-as-judge) → a versioned eval set → regression tests on every prompt or model change. "Did the generated drawing match the concept spoken?"
- **RAG/embeddings:** chunk the curriculum content, embed it, and store it in **pgvector** (simplest when Postgres already exists), Qdrant or Pinecone. Plan for re-embedding when the model changes (version the embedding columns), plus hybrid search (BM25 + vector).
- **Latency budgets for real-time AI:** speech → text (streaming STT) → LLM → drawing must feel live (under about 1 s to the first visible output). Data infrastructure supports this with per-stage timing metrics and percentile dashboards (p50/p95/p99, never only averages).
- **Cost tracking:** tokens and dollars per session or per user, alerts on spikes, caching of repeated prompts.
- The zoomcamp has a little on RAG in `02-workflow-orchestration/16-retrieval-augmented-generation.md`.

### 2.5 Batch, orchestration and analytics (`02-`, `03-`, `04-` modules)
- **Product analytics:** daily active learners, session length, the drop-off funnel, "time to first drawing," learning outcomes.
- **Stack for a small team:** events → object storage → warehouse (BigQuery, ClickHouse, or MotherDuck/DuckDB) → dbt → Metabase. Orchestrate with Dagster, Kestra or plain cron + GitHub Actions to start. Fight over-engineering.
- **Idempotency, backfills, partition overwrite, dbt incremental models with a lookback:** see `DEEP_DIVE_04_ANALYTICS_ENGINEERING.md`.

### 2.6 Platform and operations ("infrastructure" is in the title)
- **IaC:** Terraform with remote state, per-environment configs, plan in CI (`01-docker-terraform/`).
- **Containers:** Docker multi-stage builds. Kubernetes vs. ECS/Cloud Run: for a 10-person team, **managed containers (Cloud Run/ECS Fargate) are usually right**. Say when you'd move to Kubernetes: many services, custom networking, GPU scheduling.
- **WebSocket infrastructure specifics:** sticky sessions or a pub/sub backplane (Redis) so users of the same session on different servers stay in sync. Graceful draining on deploy. Load balancer idle timeouts.
- **Observability:** metrics (Prometheus/Grafana or Datadog), structured logs, traces (OpenTelemetry). Pipeline SLOs: freshness, completeness (events received vs. sent, comparing client counters with server counts), and error rate.
- **Security and privacy (important for education):** students may be **minors**. Apply data minimisation, encryption at rest and in transit, least-privilege IAM, PII separated from behaviour data, retention limits, and a right-to-deletion process. Regulation: **UAE PDPL**, plus GDPR or COPPA/FERPA if they sell to EU or US schools. Mention that voice recordings are sensitive.
- **Cost:** a startup cares a lot. Right-size, use spot instances for batch jobs, set object storage lifecycles, use budget alerts, prefer serverless or scale-to-zero.

---

## 3. System design: "Design the data infrastructure for Flow" (practice this one)

**Clarify first (say these out loud):**
- Scale now and in 12 months: concurrent sessions, events/sec per session (strokes are bursty, maybe 10–50/s while drawing), number of users.
- Consumers: session replay? teacher dashboards? product analytics? model evaluation/training?
- Latency needs: live sync (ms, hot path) vs. dashboards (minutes) vs. analytics (hourly or daily).
- Privacy: minors? where is the data stored (UAE residency)? can audio be retained?
- Team: who will operate it? (probably you)

**Architecture (draw it):**
```
Clients (web/tablet)
   │ WebSocket / WebRTC
   ▼
Realtime servers ──► Redis pub/sub (live sync between servers; hot path, not stored)
   │  async, fire-and-forget
   ▼
Event queue (Kinesis / Pub/Sub / Redpanda)  ── key = session_id
   ├─► Stream consumer ─► Postgres: session state + snapshots (replay)
   ├─► Firehose/consumer ─► Object storage, raw Parquet (date/event_type)  ◄── audio blobs (S3, lifecycle)
   │                            │
   │                            ▼
   │                      Warehouse (BigQuery / ClickHouse) ─► dbt ─► Metabase / teacher dashboards
   └─► AI call logs ─► LLM observability store ─► eval datasets ─► prompt/model regression tests

Cross-cutting: Terraform · CI/CD · OpenTelemetry + alerts · IAM / encryption / retention & deletion jobs
```

**Deep dives they might push on:**
1. **Session replay correctness:** ordering per session, snapshot + log, idempotent dedup on `event_id`, handling clients that reconnect and resend buffered events.
2. **Late or offline events:** a tablet loses Wi-Fi and syncs 2 minutes later. Event time vs. server time, a lookback window in the downstream aggregations (module 07 late events, module 04 incremental lookback).
3. **Schema evolution:** a `schema_version` field, a registry or JSON Schema validation at ingestion, additive changes only, a dead-letter queue for events that fail validation.
4. **Deletion requests:** a user ID → all stores map; deletes in Postgres; on the lake, either partition by user or tenant, or use Iceberg/Delta row deletes; audio removed by prefix. Show that you thought about this at design time.
5. **Evolution path:** v0 = Postgres + nightly export to the warehouse. v1 = a managed queue + lake. v2 = stream processing (Flink) only if real-time analytics becomes a product feature, e.g. a live teacher dashboard showing class engagement.

**Close with trade-offs:** "I'm choosing managed services over Kafka on Kubernetes because there's a team of one or two to operate it. The cost is some lock-in and less control. I'd revisit at about X events per second or when we need multi-consumer replay."

---

## 4. Likely hands-on tasks
- **SQL:** sessions per user, session duration from start and end events (`lead()` or min/max per session), funnel drop-off, p95 latency with `percentile_cont`, deduplicating on `event_id` (see `INTERVIEW_PREP.md` §4).
- **Python:** an idempotent consumer (read batch → dedup → upsert → commit offset *after* the write), retry with backoff, chunked writes of Parquet to S3, a simple FastAPI ingestion endpoint with validation (pydantic).
- **Debugging scenario:** "Dashboards show 20% fewer sessions since Tuesday's deploy." Walk through it: compare client-sent counters with server-received events, check queue consumer lag and the dead-letter queue, check for schema validation rejects, then the deploy diff. The fix should come with a completeness alert.
- **Postgres scenario:** "Event table queries got slow at 200M rows." Look at `EXPLAIN ANALYZE`, check indexes (BRIN on time), partition by month, move analytics off the primary, check vacuum and bloat.

---

## 5. Your intro (adapt) and the stories to pick

> "I'm a data engineer focused on building pipelines and the platforms underneath them: batch and streaming ingestion, warehouses and dbt modelling, and the infrastructure (Docker, Terraform, cloud) to run them reliably. Recently I ___ [one win with a number]. Flow interests me because real-time collaboration plus AI produces exactly the kind of data that's hard to get right: high-frequency, ordered per session, privacy-sensitive, and directly useful for improving the model. In a small team I'm comfortable owning everything from event schema to dashboards."

**Prepare three short STAR stories:**
1. **Built something from zero:** chose simple tools and shipped quickly. (Small startups love this.)
2. **A production incident:** found the root cause, fixed it, and added monitoring so it couldn't recur.
3. **A trade-off you pushed back on:** e.g. no Kafka yet, a managed service instead, and the cost or time it saved.

---

## 6. Cheat sheet (read tomorrow morning)
- **Hot path ≠ analytics path.** Analytics is asynchronous and never blocks the live canvas.
- **Key by `session_id`** → ordering per session. **`event_id`** → idempotent dedup. **`client_ts` + `server_ts`**.
- **Snapshot + event log** = replay and recovery.
- **Raw Parquet in object storage, partitioned by date** → warehouse → dbt → BI.
- **Postgres:** partition by time, BRIN, PgBouncer, replicas for reads, tested PITR.
- **LLM data:** log every call (tokens, latency p95, cost, prompt version) → eval sets → regression tests.
- **Privacy:** minors and voice data → minimise, encrypt, set retention, design deletion in from the start.
- **Small team** → managed and serverless first. Say *when* you'd move to the heavier option.
- Every answer gets a *why*, a *trade-off*, and *what you'd do next*.

---

## 7. Questions to ask them
1. What does the data stack look like today, and what's the most painful part?
2. What does "data infrastructure" cover here: product analytics, AI and evaluation data, the real-time backend, or all of them?
3. How do you evaluate whether the AI drawings are good today? Is there an eval pipeline?
4. Who are the users today: schools, individual learners, enterprise studio clients? Any data residency or child-privacy requirements?
5. How is engineering time split between Flow and studio client work?
6. What would make the first 90 days a success for this hire?

---

### Sources
- [Crunchbase — Apekso](https://www.crunchbase.com/organization/apekso)
- [Dealroom — Apekso](https://app.dealroom.co/companies/apekso)
- [Briter — Apekso](https://www.briter.co/companies/apekso)
- [Apekso — About](https://apekso.com/about) (not readable from this session; check it yourself)
