# Lead Data Engineer Interview Prep — Friday Oct 30, 3 PM

Built on top of this repo (DE Zoomcamp). Each section points at the module/files to review.

A **lead** interview checks four things, roughly equally:
1. **Technical depth.** Can you explain *why*, not just *how*: trade-offs, failure modes, cost.
2. **System design.** Can you design a platform end to end and defend your choices?
3. **Hands-on.** Live SQL, usually Python too, sometimes reading or debugging a pipeline.
4. **Leadership.** Ownership, mentoring, standards, saying no, handling incidents, working with stakeholders.

---

## 1. Four-day plan

| Day | Focus | Output |
|---|---|---|
| **Mon (today)** | Read this doc. Do the SQL drills (§4) without looking at the answers. Draft 6 STAR stories (§6). | Stories in bullet form |
| **Tue** | Modules 01–04: Docker, orchestration, warehouse, dbt. Answer every question in §3 out loud. | 60–90 s answer for each question |
| **Wed** | Modules 06–07: Spark, streaming. Two timed system-design runs (§5), 35 min each, drawn on paper or Excalidraw. | 2 diagrams |
| **Thu** | Research the company: stack, product, data scale. Polish stories. Do a mock interview: say answers out loud or record yourself. | Questions for them (§7) |
| **Fri AM** | Light review of the cheat sheet (§8) only. Test your setup. No new material. | — |

---

## 2. Your 2-minute intro (write it, then say it out loud 5×)

Structure: **current role → scope (team size, data volume, stack) → one big win with a number → why this role.**

> "I'm a data engineer with X years building batch and streaming pipelines. In my current role I own ___, processing ~__ GB/day on ___ (stack). Recently I led ___, which cut ___ by __%. I've mentored __ engineers and set up ___ (CI, testing, data contracts). I want a lead role because ___."

---

## 3. Technical questions by module

### Module 01 — Docker & Terraform (`01-docker-terraform/`)
Review: `09-docker-compose.md`, `12-terraform-overview.md`, `terraform/`

- **Image vs. container?** An image is an immutable, layered template. A container is a running instance of it with a writable layer.
- **How do you make Docker images smaller and builds faster?** Use slim or distroless base images and multi-stage builds. Put the layers that change least first, e.g. copy `requirements.txt` and install dependencies *before* copying the code. Add a `.dockerignore`. Pin versions.
- **Why do containers in docker-compose reach each other by service name?** Compose creates a shared network with built-in DNS. `localhost` inside a container means *that container*. That's why the ingestion script connects to `pgdatabase`, not `localhost`.
- **Terraform state: what is it and how do you manage it on a team?** State maps your config to real resources. On a team, keep it in a remote backend (a GCS or S3 bucket) with locking and versioning. Never commit it to git, because it can contain secrets. Split state by environment or domain to limit the blast radius.
- **`plan` vs. `apply`, and how do you run Terraform safely?** Run `plan` in CI on every PR and post the output for review. Run `apply` only from main through the pipeline. Reuse code with modules and `tfvars` per environment.

### Module 02 — Orchestration (`02-workflow-orchestration/`)
Review: `08-scheduling-and-backfills.md`, `09-etl-vs-elt.md`

- **ETL vs. ELT?** ELT loads raw data into the warehouse and transforms it there (dbt). Cheap storage and elastic compute make this the default today. Choose ETL when you must mask PII before landing, or when the target can't do the transforms.
- **What makes a pipeline idempotent, and why does a lead care?** Running it twice for the same input gives the same result. Techniques: partition overwrite per run date (`DELETE`+`INSERT` or `MERGE` on a key), deterministic file paths like `.../date=2026-10-30/`, no `now()` inside logic (use the scheduled logical date). Idempotency is what makes retries and backfills safe.
- **How do you backfill 2 years of data?** Use parameterised runs by logical date and limited concurrency so you don't overload the source or the warehouse. Make the runs idempotent and backfill to a staging table first. Validate row counts and checksums against the source, then swap or merge. Kestra and Airflow both support backfills natively.
- **Retries vs. alerting?** Retry transient failures (network, rate limits) with exponential backoff. Fail fast on data or logic errors. Alert on SLA misses and failures, not on every retry.
- **Kestra vs. Airflow vs. Dagster?** Airflow: task-centric, Python DAGs, biggest ecosystem. Dagster: asset-centric, strong lineage and typing. Kestra: declarative YAML, event-driven, easy for non-Python teams. Have an opinion and justify it by team skills and the use case.

### Module 03 — Data Warehouse / BigQuery (`03-data-warehouse/`)
Review: `02-partitioning-vs-clustering.md`, `03-bigquery-best-practices.md`, `04-internals-of-bigquery.md`, `big_query.sql`

- **Partitioning vs. clustering?** Partitioning splits a table into segments, usually by date or ingestion time. Queries that filter on the partition column skip whole partitions, and the cost estimate is known up front. Clustering sorts data inside partitions by up to 4 columns, which helps filters and aggregations on those columns through block pruning. Use clustering when partitions would be too small (<1 GB) or the column has high cardinality.
- **Why is BigQuery fast?** Columnar storage (Capacitor) on Colossus, compute separated from storage (Dremel), a tree architecture that fans queries out to thousands of slots, and the Jupiter network.
- **How do you cut warehouse cost?** Never `SELECT *`. Always filter on the partition column (set `require_partition_filter`). Materialise expensive intermediate results. Prefer incremental models to full refreshes. Use slot reservations for predictable spend. Watch query cost per user/job in `INFORMATION_SCHEMA.JOBS`.
- **External vs. native tables?** External tables read files in GCS: no storage cost and always fresh, but slower, with no cost estimate and limited pruning. Native tables are faster and support partitioning and clustering.
- **Star schema: fact vs. dimension? SCD types?** Facts are events or measures (`fct_trips`). Dimensions are descriptive context (`dim_zones`). SCD1 overwrites history. SCD2 adds a new row with `valid_from`, `valid_to`, `is_current` (dbt snapshots do this). Know when denormalised wide tables beat a star schema in a columnar warehouse.

### Module 04 — dbt (`04-analytics-engineering/taxi_rides_ny/`)
Review: `models/staging → intermediate → marts`, `models/marts/fct_trips.sql`, `macros/`, `09-dbt-tests.md`

- **Walk me through your layering.** Staging: 1:1 with sources; rename, cast, light cleaning (`stg_yellow_tripdata`). Intermediate: unions, joins, business logic, dedup (`int_trips_unioned`, `int_trips`). Marts: consumer-facing facts and dimensions (`fct_trips`, `dim_zones`) plus reporting tables (`fct_monthly_zone_revenue`).
- **How does `fct_trips` work incrementally?** It uses `materialized='incremental'`, `incremental_strategy='merge'`, `unique_key='trip_id'`. New and changed rows are merged on `trip_id`. `on_schema_change='append_new_columns'` handles new columns. Know the risk: late-arriving data. Use a lookback window (`where pickup_datetime > max(...) - interval 3 day`) and schedule periodic full refreshes.
- **How is `trip_id` generated?** `dbt_utils.generate_surrogate_key` on (vendor_id, pickup_datetime, pickup_location_id, service_type). Follow-up: *"What if two trips collide?"* Deduplicate with `row_number()` or add more columns to the key, and test with `unique`.
- **Testing strategy?** Generic tests (`unique`, `not_null`, `accepted_values`, `relationships`), singular SQL tests, source `freshness`, and `dbt_expectations` for distributions and ranges. Run `dbt build --select state:modified+` in CI (slim CI). Use `severity: warn` vs. `error` deliberately.
- **Seeds vs. sources vs. macros?** Seeds are small static CSVs in git (`payment_type_lookup`). Sources are raw tables declared in YAML, with freshness checks and lineage. Macros are reusable Jinja (`get_trip_duration_minutes`, `safe_cast`) that keep logic DRY and portable across warehouses.

### Module 05 — Data platforms (`05-data-platforms/`)
- **How do you evaluate an all-in-one tool (Bruin, Databricks, Fivetran plus dbt) against composing your own stack?** Weigh team size, vendor lock-in, cost at scale, lineage and governance needs, and time to value. A lead decides based on total cost of ownership, not hype.

### Module 06 — Spark (`06-batch/`)
Review: `08-anatomy-of-a-spark-cluster.md`, `09-groupby-in-spark.md`, `10-joins-in-spark.md`, `code/06_spark_sql.py`

- **Driver, executors, partitions, tasks?** The driver plans the job and builds the DAG. Each action triggers a job. Jobs split into stages at shuffle boundaries, and each stage runs one task per partition on the executors.
- **Transformations vs. actions? Narrow vs. wide?** Transformations are lazy. Actions trigger execution. Narrow transformations (`map`, `filter`) need no data movement. Wide ones (`groupBy`, `join`, `distinct`) shuffle data, which is expensive.
- **How does a groupBy work?** A partial aggregation runs per partition, then a shuffle by key (exchange), then the final aggregation. That's why `reduceByKey` beats `groupByKey`.
- **Join strategies?** Broadcast hash join when one side is small (under `spark.sql.autoBroadcastJoinThreshold`, or force it with `broadcast()`): no shuffle of the big side. Sort-merge join for two large tables. In the taxi example, joining zones to trips is a broadcast.
- **Data skew: how do you detect and fix it?** Symptoms: one task runs far longer than the rest in the Spark UI, or spills. Fixes: AQE skew-join handling (`spark.sql.adaptive.enabled`), salting the hot key, broadcasting the small side, filtering nulls or hot keys and handling them separately.
- **`repartition` vs. `coalesce`?** `repartition(n)` does a full shuffle and produces even partitions. `coalesce(n)` only reduces the count without a shuffle, which is cheap but can be uneven. Use it to avoid writing thousands of small files.
- **Small files problem?** Many tiny files slow down listing and reads. Fix by coalescing on write, compacting, using sensible `partitionBy` granularity, or using table formats (Delta/Iceberg `OPTIMIZE`).
- **Parquet vs. CSV?** Parquet is columnar, compressed, carries a schema, and supports predicate pushdown and column pruning. CSV has none of these.
- **Dataproc vs. local cluster? Spark → BigQuery?** Know the `spark-submit` flow in `06_spark_sql_big_query.py` (writing through the BigQuery connector with a temporary GCS bucket).

### Module 07 — Streaming (`07-streaming/`)
Review: `09-offsets-earliest-vs-latest.md`, `10-aggregation-with-tumbling-windows.md`, `11-late-events-and-upserts.md`, `12-understanding-window-types.md`, `theory/`

- **Kafka basics?** Topic → partitions (the unit of ordering and parallelism) → offsets. A consumer group shares a topic's partitions, so you can have at most one active consumer per partition. Messages with the same key go to the same partition, which gives you ordering per key. Redpanda is Kafka API-compatible, written in C++, with no ZooKeeper or JVM.
- **Delivery semantics?** At-most-once, at-least-once (the default; make sinks idempotent or upsert), exactly-once (idempotent producer plus transactions, or Flink checkpoints with two-phase-commit sinks).
- **`earliest` vs. `latest`?** This only applies when the group has no committed offset: replay everything, or start from new messages only.
- **Event time vs. processing time; watermarks?** Event time is when the event happened. A watermark asserts "no events older than T are expected", which lets windows close. Allowed lateness and upserts into the sink (e.g. `ON CONFLICT DO UPDATE` in Postgres) handle stragglers.
- **Window types?** Tumbling: fixed size, no overlap (trips per 5 min). Hopping/sliding: fixed size, overlapping. Session: closes after an inactivity gap.
- **Why Flink over a Python consumer?** Managed state, checkpoints and exactly-once, event-time semantics, scalability, SQL API.
- **Batch or streaming? (a lead question)** Ask what latency the business actually needs. Streaming costs more to build and run. Micro-batching every 5–15 minutes often covers "real-time" dashboards.

### Cross-cutting topics a lead must own (not deep in the course; prepare these)
- **Data quality and observability.** Tests at ingestion and transform, freshness/volume/schema checks, anomaly alerts, data contracts with producers.
- **Schema evolution.** Additive changes are fine. Breaking changes need versioning, a contract, or a schema registry (Avro/Protobuf with compatibility modes).
- **CDC.** Debezium reads the DB log → Kafka → warehouse `MERGE`. Mention why log-based CDC beats `updated_at` polling: it captures deletes and puts less load on the source.
- **Lakehouse table formats.** Iceberg, Delta, Hudi: ACID, time travel, schema evolution, compaction.
- **Governance and security.** IAM least privilege, PII tagging, column-level security and masking, GDPR deletes.
- **CI/CD for data.** Linting (sqlfluff), unit tests, dbt slim CI, environment separation (dev/stage/prod), infrastructure as code.
- **Cost (FinOps).** Cost per pipeline or team, budgets and alerts, killing unused tables and jobs.

---

## 4. SQL drills (taxi data; do these without looking first)

Assume `trips(trip_id, vendor_id, pickup_datetime, dropoff_datetime, pickup_location_id, dropoff_location_id, fare_amount, total_amount)` and `zones(location_id, borough, zone)`.

**Q1. Top 3 pickup zones by revenue in each borough.**
<details><summary>Answer</summary>

```sql
with zone_rev as (
  select z.borough, z.zone, sum(t.total_amount) as revenue
  from trips t join zones z on t.pickup_location_id = z.location_id
  group by 1, 2
)
select * from (
  select *, dense_rank() over (partition by borough order by revenue desc) as rnk
  from zone_rev
) where rnk <= 3;
```
Follow-up: `rank` vs. `dense_rank` vs. `row_number` on ties.
</details>

**Q2. Deduplicate trips: keep the latest record per `trip_id` (by an `ingested_at` column).**
<details><summary>Answer</summary>

```sql
select * except(rn) from (
  select *, row_number() over (partition by trip_id order by ingested_at desc) as rn
  from trips_raw
) where rn = 1;
-- BigQuery shortcut: ... QUALIFY row_number() over (...) = 1
```
</details>

**Q3. Daily revenue with a 7-day rolling average and day-over-day % change.**
<details><summary>Answer</summary>

```sql
with daily as (
  select date(pickup_datetime) as d, sum(total_amount) as rev
  from trips group by 1
)
select d, rev,
  avg(rev) over (order by d rows between 6 preceding and current row) as rev_7d_avg,
  safe_divide(rev - lag(rev) over (order by d), lag(rev) over (order by d)) as dod_pct
from daily;
```
Follow-up: what happens when days are missing? Generate a date spine and left join to it.
</details>

**Q4. Gaps and islands: for each vendor, find streaks of consecutive days with at least one trip.**
<details><summary>Answer</summary>

```sql
with days as (select distinct vendor_id, date(pickup_datetime) as d from trips),
grp as (
  select *, date_sub(d, interval row_number() over (partition by vendor_id order by d) day) as island
  from days
)
select vendor_id, min(d) as start_d, max(d) as end_d, count(*) as len
from grp group by vendor_id, island order by len desc;
```
</details>

**Q5. Data quality: find trips with negative fares, dropoff before pickup, or duration over 24 h, as % of total by month.**
<details><summary>Answer</summary>

```sql
select date_trunc(date(pickup_datetime), month) as m,
  countif(fare_amount < 0 or dropoff_datetime < pickup_datetime
          or timestamp_diff(dropoff_datetime, pickup_datetime, hour) > 24) / count(*) as bad_pct
from trips group by 1 order by 1;
```
</details>

**Q6. SCD Type 2: given `zones_current` (today's snapshot) and `dim_zones_scd`, write the logic for changed rows.**
<details><summary>Answer (outline)</summary>

1. Find changed keys: join current to `dim_zones_scd where is_current` on `location_id` where any attribute differs (or a hash of the attributes differs).
2. `UPDATE` those rows: `set valid_to = current_date, is_current = false`.
3. `INSERT` new versions (changed + brand-new keys) with `valid_from = current_date, valid_to = null, is_current = true`.

In dbt, this is just a `snapshot` with `strategy='check'` or `'timestamp'`.
</details>

**Also be ready for:** `MERGE` syntax, `LEFT JOIN` + `WHERE` turning into an inner join, `COUNT(col)` vs. `COUNT(*)` with NULLs, `UNION` vs. `UNION ALL`, and reading an `EXPLAIN` or query plan.

**Python quick hits:** write a generator that reads a large CSV in chunks (`pd.read_csv(chunksize=...)`, like the ingestion script in module 01), retry with backoff, paginate an API, and explain why you'd use `uv`/`venv`.

---

## 5. System design

### Framework (use it out loud, every time)
1. **Clarify (5 min).** Who uses the data and how fresh must it be (SLA)? Volume and velocity? Sources? Correctness needs (finance = exactly once)? Budget, cloud, team skills? Compliance or PII?
2. **High-level diagram.** Sources → ingestion → storage (raw/bronze → clean/silver → marts/gold) → serving → consumers. Orchestration, observability, governance run across all of it.
3. **Deep dive on 2 parts** they care about (often ingestion and modelling, or streaming state).
4. **Failure modes.** Late data, duplicates, schema change, backfill, source outage, reprocessing.
5. **Operate it.** Monitoring, alerting, data quality, CI/CD, cost, on-call, ownership.
6. **Trade-offs and what you'd do in v2.**

### Practice prompt A — "Design an analytics platform for a ride-hailing company" (batch; maps to the full course)
- Sources: app Postgres (trips, users), payments API, zone CSVs.
- Ingestion: CDC (Debezium → Kafka) for Postgres; scheduled API extraction (dlt/Kestra) into GCS raw, as partitioned Parquet.
- Warehouse: BigQuery. Raw external/native tables → dbt staging/intermediate/marts. `fct_trips` partitioned by `pickup_date` and clustered by `pickup_location_id`, merged incrementally.
- Orchestration: Kestra/Airflow. Daily, idempotent per logical date. Backfills through parameters.
- Quality: dbt tests, freshness checks, row-count reconciliation against the source.
- Serving: Looker/Metabase on marts. Reverse ETL if needed.
- Infra: Terraform, Docker, CI with dbt slim CI. Separate dev/prod projects.
- Discuss: PII masking of rider data, cost controls, SLA (dashboards fresh by 7 AM), ownership model.

### Practice prompt B — "Real-time surge pricing / live trips dashboard" (maps to module 07)
- Driver app events → Kafka (keyed by `zone_id`, partitions sized for peak throughput).
- Flink: event-time tumbling or hopping windows (e.g. 1-minute demand per zone), watermark of ~30 s, allowed lateness, checkpointing to GCS.
- Sinks: Redis or Postgres upsert for the live dashboard and pricing service. Raw events also go to the lake for batch reconciliation (Kappa-style replay, or Lambda with batch correction).
- Discuss: exactly-once vs. idempotent upsert, hot zones (skew → finer keys), reprocessing after bugs (reset offsets, replay from the lake), monitoring consumer lag.

### Other prompts to sketch quickly
- Migrate an on-prem Hadoop/Oracle warehouse to the cloud (phased, dual-run, reconciliation, cut-over plan).
- Ingest 500 third-party APIs with different schemas (config-driven framework, schema registry or contracts, dead-letter queue).
- A dashboard number is "wrong". How do you investigate? (lineage → recent deploys → source changes → data diffs, then add a test so it can't happen again).

---

## 6. Leadership and behavioral (as much weight as the technical part for a lead)

Prepare **6 STAR stories** (Situation, Task, Action, Result with a number). Each one can cover several of these questions:

| Theme | Typical question |
|---|---|
| Technical leadership | "Tell me about an architecture decision you drove. What alternatives did you reject?" |
| Incident / failure | "A pipeline broke production data. What happened and what did you change afterwards?" (blameless post-mortem, prevention) |
| Mentoring | "How did you grow a junior engineer?" (pairing, code review standards, stretch tasks) |
| Conflict / stakeholders | "A stakeholder wanted X by Friday and it wasn't feasible." (trade-offs, a smaller scope, data-driven pushback) |
| Standards / quality | "How did you raise the engineering bar?" (CI, testing, PR templates, docs, data contracts) |
| Prioritisation | "How do you balance tech debt against features?" (e.g. 20% capacity, tie debt to business risk or cost) |
| Ambiguity | "You joined and there was no data platform. What did you do in your first 90 days?" |

**First 90 days answer (very common for leads):**
- **Days 0–30:** listen. Map pipelines, owners, pain points, costs. Meet stakeholders. Ship one small fix.
- **Days 31–60:** quick wins (alerting, tests on critical models, cost cleanup). Write a roadmap.
- **Days 61–90:** start the bigger initiative, set team practices, and agree on success metrics (SLA adherence, incident count, cost).

---

## 7. Questions to ask them

1. What does the current data stack look like, and what's the biggest pain point right now?
2. What would success look like for this role at 6 and 12 months?
3. How big is the team, and what is the split between hands-on work and people leadership?
4. How are data quality and incidents handled today? Is there on-call?
5. Who are the main data consumers, and how are priorities set between them?
6. Build vs. buy: how does the team make tooling decisions?

---

## 8. Friday cheat sheet (read Friday morning only)

- **Idempotent + partition overwrite + logical date** → safe retries and backfills.
- **Partition** = skip segments (date). **Cluster** = sort within (filter/group columns).
- **Incremental merge** on a unique key + **lookback** for late data.
- **Spark**: stage boundary = shuffle. Broadcast small tables. AQE/salting for skew. Coalesce before write.
- **Kafka**: partition = ordering + parallelism. Key → partition. At-least-once + idempotent sink.
- **Flink**: event time + watermark closes windows. Late events → upsert.
- **Design**: clarify → diagram → deep dive → failures → operate → trade-offs.
- **Lead**: every answer has a *why*, a *trade-off*, and a *number*.

**Logistics:** test your camera, mic and screen share by 2:30 PM. Have water, a pen and paper, and this doc closed. It's fine to think out loud and to ask clarifying questions.
