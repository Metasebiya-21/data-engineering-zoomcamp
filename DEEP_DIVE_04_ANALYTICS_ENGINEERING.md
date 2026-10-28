# Deep Dive: Module 04 — Analytics Engineering (dbt)

Project: `04-analytics-engineering/taxi_rides_ny/`

How to use this: read §1–2 so you can *explain* the project, then §3 so you can *critique* it. §3 is what separates a lead from a mid-level engineer. Then drill §4 out loud.

---

## 1. The project in 60 seconds (say this if asked "walk me through a dbt project")

```
sources (BigQuery: nytaxi.green_tripdata, nytaxi.yellow_tripdata)      seeds (CSV in git)
        │                                                                │
        ▼  views                                                         │
stg_green_tripdata   stg_yellow_tripdata     ← rename, cast, filter null vendor
        │                    │
        └──────┬─────────────┘
               ▼  table
       int_trips_unioned                     ← align schemas, add service_type, UNION ALL
               ▼  table
           int_trips          ◄── payment_type_lookup (seed)
               │                              ← surrogate key, dedup (QUALIFY), enrich
               ▼  incremental (merge)
           fct_trips          ◄── dim_zones ◄── taxi_zone_lookup (seed)
            │       │
            ▼       ▼
   dim_vendors   fct_monthly_zone_revenue    ← reporting mart for dashboards
```

> "Raw TLC trip data lands in BigQuery. Staging models are views, one per source: they standardise names and types, so `tpep_pickup_datetime` and `lpep_pickup_datetime` both become `pickup_datetime`. The intermediate layer unions green and yellow trips, generates a surrogate key, deduplicates, and enriches from seeds. Marts are a star schema: `fct_trips` is incremental with a merge on `trip_id` and an enforced contract, joined to `dim_zones`, with a monthly revenue mart for BI. Quality is covered by generic tests, a composite-key test, relationship tests, and source freshness."

---

## 2. Layer-by-layer: what each piece does and *why*

### 2.1 Project config: `dbt_project.yml`, `packages.yml`
- **Materialization defaults by folder:** staging = `view`, intermediate = `table`, marts = `table`. Folder-level `+materialized` keeps models consistent without repeating config in each one.
  - *Why views for staging?* They're cheap, always fresh, and add no storage. They're 1:1 with sources, so materializing them would only duplicate raw data.
- **`require-dbt-version`** pins the dbt version so builds are reproducible across the team and CI.
- **`vars: dev_start_date / dev_end_date`** are meant for sampling in dev (see issue #6 in §3).
- **Packages:** `dbt_utils` (surrogate keys, generic tests) and `codegen` (generates source and model YAML boilerplate). The `package-lock.yml` pins exact versions.

### 2.2 Sources: `models/staging/sources.yml`
- `source('raw', 'yellow_tripdata')` instead of a hard-coded table name gives you **lineage**, **freshness checks**, and one place to change the location.
- **Multi-adapter Jinja:** `database`/`schema` switch on `target.type` (BigQuery vs. DuckDB), and the GCP project comes from `env_var('GCP_PROJECT_ID')`. No secrets live in code.
- **Freshness:** `warn_after 24h`, `error_after 48h`, using `loaded_at_field` (see issue #5 in §3).

### 2.3 Staging: `stg_green_tripdata.sql`, `stg_yellow_tripdata.sql`
- The CTE pattern `source → renamed → final select` is the dbt style guide convention.
- Explicit `cast(...)` on every column, so downstream models get stable types.
- Green uses the project macro `safe_cast` (BigQuery `SAFE_CAST` returns NULL instead of failing on bad values). Yellow uses plain `cast`. Expect the question "*why the difference?*" Green's `ratecodeid`, `trip_type` and `payment_type` contain dirty values. A lead would standardise on one approach.
- `where vendorid is not null` is a data quality filter, documented in the source description.
- Dev sampling: `{% if target.name == 'dev' %} where pickup_datetime ...` means fast, cheap dev builds and full data in prod.

### 2.4 Intermediate: `int_trips_unioned.sql`, `int_trips.sql`
- **Union of different schemas:** yellow has no `trip_type` or `ehail_fee`, so they're filled with constants (`1`, `0`) using explicit casts so the `UNION ALL` types line up. A `service_type` literal says which source each row came from.
- **`UNION ALL` vs. `UNION`:** `UNION` deduplicates, which is an expensive sort or hash and hides problems. Use `UNION ALL` and deduplicate explicitly.
- **Surrogate key:** `dbt_utils.generate_surrogate_key([vendor_id, pickup_datetime, pickup_location_id, service_type])` is an MD5 of the columns cast to string, with NULLs replaced by a placeholder. It's deterministic, so the same trip gets the same key on every run, which is what makes the `merge` work.
- **Dedup:** `qualify row_number() over (partition by <key cols> order by dropoff_datetime) = 1`.
- **Enrichment:** `left join` to the `payment_type_lookup` seed with `coalesce(payment_type, 0)` → 'Unknown'. A left join never drops trips. Unknowns are labelled, not lost.

### 2.5 Marts
- **`fct_trips`:** incremental, `incremental_strategy='merge'`, `unique_key='trip_id'`, `on_schema_change='append_new_columns'`. Zone names are **denormalised** into the fact through two left joins to `dim_zones` (pickup and dropoff are a *role-playing dimension*). Trip duration comes from the cross-database macro `get_trip_duration_minutes` → `dbt.datediff`.
  - **Model contract** (`contract: enforced: true` in `marts/schema.yml`): every column's `data_type` is declared, and dbt fails the build if the SQL output doesn't match. Contracts on incremental models require `on_schema_change` to be `append_new_columns` or `fail`. That's why this model uses `append_new_columns`.
- **`dim_zones`:** a pass-through from the seed. Wrapping a seed in a model gives you a stable interface you can extend later.
- **`dim_vendors`:** `select distinct vendor_id` from `fct_trips`, plus the `get_vendor_data` macro (a Jinja dict that compiles to a `CASE`). See issue #7 in §3.
- **`fct_monthly_zone_revenue`:** a pre-aggregated reporting table keyed by (zone, month, service_type), tested with `dbt_utils.unique_combination_of_columns`. Dashboards hit a small table instead of scanning billions of rows.

### 2.6 Tests in place
| Where | Tests |
|---|---|
| staging | `not_null` on vendor_id, pickup_datetime |
| int_trips | `unique` + `not_null` on trip_id, `accepted_values` on service_type |
| fct_trips | `unique`/`not_null` trip_id, `relationships` pickup/dropoff → `dim_zones.location_id`, enforced contract |
| reporting | composite-key uniqueness, `not_null` |
| seeds | `unique`/`not_null` on payment_type |
| sources | freshness |

The `arguments:` nesting under each test, and `flags: require_generic_test_arguments_property: true`, are the newer dbt syntax (1.10+). Mention that if asked about upgrading dbt.

---

## 3. Code review: 9 issues a lead should spot (your differentiator)

If they ask "*what would you improve?*" or hand you a dbt project to review, this is the answer. For each issue: **problem → impact → fix**.

### #1 Incremental filter misses late-arriving data ⚠️ (most important)
```sql
where trips.pickup_datetime > (select max(pickup_datetime) from {{ this }})
```
- **Problem:** a trip with an older `pickup_datetime` that lands after the last run is skipped forever. TLC also republishes corrected months. Using `merge` on `unique_key` suggests updates were intended, but this filter only ever lets *new* rows through, so updates never happen.
- **Also:** one bad future timestamp (TLC data has trips dated 2088) pushes `max()` forward, and every later run silently loads nothing.
- **Fix:** a lookback window, partitioning, and a cap on future dates:
```sql
{{ config(
    materialized='incremental',
    unique_key='trip_id',
    incremental_strategy='merge',
    partition_by={'field': 'pickup_datetime', 'data_type': 'timestamp', 'granularity': 'day'},
    cluster_by=['pickup_location_id', 'service_type'],
    incremental_predicates=["DBT_INTERNAL_DEST.pickup_datetime >= timestamp_sub(current_timestamp(), interval 7 day)"],
    on_schema_change='append_new_columns'
) }}
...
{% if is_incremental() %}
where trips.pickup_datetime >= (
    select timestamp_sub(max(pickup_datetime), interval {{ var('lookback_days', 3) }} day) from {{ this }}
)
{% endif %}
```
Also filter `pickup_datetime <= current_timestamp()` in staging and test it.
- **Alternative on BigQuery:** `incremental_strategy='insert_overwrite'` with `partition_by` replaces whole day partitions. It's cheaper than `merge` because no join against the destination is needed, and it's naturally idempotent.
- `incremental_predicates` restricts the destination side of the MERGE so it doesn't scan the whole fact table.

### #2 The incremental fact saves little because upstream is a full rebuild
- `int_trips_unioned` and `int_trips` are `table`s rebuilt from **all history** on every run, including the window-function dedup over billions of rows. Only the final `fct_trips` step is incremental.
- **Fix:** push incrementality upstream (make `int_trips` incremental with the same lookback), or make the intermediates `ephemeral`/views so the fact's incremental filter is pushed down into the source scan. **Lesson:** an incremental model is only as cheap as its most expensive upstream model.

### #3 No partitioning or clustering on the big fact
- On BigQuery, every dashboard query and every `unique` test scans the full table.
- **Fix:** `partition_by` day on `pickup_datetime` and `cluster_by` the common filter columns (shown in #1). Consider `require_partition_filter` for ad-hoc users.

### #4 The surrogate key and dedup can silently drop real trips
- The key is (vendor, pickup second, pickup zone, service). Two genuinely different trips from the same vendor and zone in the same second collide, and `QUALIFY` keeps one and drops the other **without any signal**.
- The `order by dropoff_datetime` tiebreaker isn't deterministic when dropoffs are equal, so different runs can keep different rows (and the `merge` then flip-flops values).
- The `unique` test on `int_trips.trip_id` is **tautological**: the `QUALIFY` guarantees it can never fail, so it gives false confidence.
- **Fix:** include more columns in the key (dropoff_datetime, dropoff_location_id, total_amount). Make the ordering fully deterministic. **Measure** what dedup removes (an audit model or a test with `warn_if: ">1000"` on the count of dropped rows) so a source problem shows up as a rising duplicate rate.

### #5 Source freshness is checked on the wrong column
- `loaded_at_field: tpep_pickup_datetime` measures when the *trip happened*, not when the data was *loaded*. Historical TLC data is years old, so freshness always errors. And a single future-dated row would make the data look perfectly fresh.
- **Fix:** have the loader add an ingestion timestamp (`_loaded_at`, or `_dlt_load_id` with dlt) and point `loaded_at_field` at it.

### #6 Dev sampling ignores the project vars
- `dbt_project.yml` defines `dev_start_date`/`dev_end_date`, but both staging models hard-code `'2019-01-01'` and `'2019-02-01'`. The vars do nothing.
- **Fix:** `where pickup_datetime >= '{{ var("dev_start_date") }}' and pickup_datetime < '{{ var("dev_end_date") }}'`. Better still, put it in one macro (`{{ dev_limit('pickup_datetime') }}`) so the logic isn't duplicated across models. You can override it with `dbt run --vars '{dev_start_date: 2020-01-01}'`.

### #7 `dim_vendors` is derived from the fact (inverted dependency)
- The dimension depends on `fct_trips`: it does a full-scan `select distinct` on every run, and creates a fact → dim edge in the DAG. Dimensions should be upstream of facts.
- The macro's `CASE` has **no `ELSE`**, so any new vendor ID (TLC has added 6 and 7) gets a NULL `vendor_name` without any error.
- **Fix:** make `vendor_lookup.csv` a seed, build `dim_vendors` from the seed, add `else 'Unknown'`, and add a `relationships` test from `fct_trips.vendor_id` to `dim_vendors`.

### #8 The reporting mart isn't portable across adapters
```sql
{% if target.type == 'bigquery' %}cast(date_trunc(pickup_datetime, month) as date)
{% elif target.type == 'duckdb' %}date_trunc('month', pickup_datetime)
{% endif %} as revenue_month
```
- On Snowflake, Postgres or Redshift, this renders an **empty expression**, which is a compile-time SQL error.
- **Fix:** use dbt's cross-database macro: `cast({{ dbt.date_trunc('month', 'pickup_datetime') }} as date)`. The project already does this correctly with `dbt.datediff` in `get_trip_duration_minutes`.
- Also: it's a full rebuild over all of `fct_trips` on every run. It could be incremental by month, with the current month re-processed. And it includes a *partial current month*, which misleads dashboards unless the data is labelled or filtered.

### #9 Smaller points (mention 1–2 to show attention to detail)
- **Denormalised zone names in an incremental fact go stale.** If `taxi_zone_lookup` renames a zone, old rows keep the old name until a `--full-refresh`. This is a trade-off: decide deliberately, or keep only IDs in the fact and join at query time or in a view.
- **The `unique` test on `fct_trips.trip_id`** scans the whole table every run. Scope it with `config: where: "pickup_datetime >= current_date - 7"`, or run full-table checks on a nightly schedule.
- **Missing business-rule tests:** `dbt_utils.expression_is_true` for `dropoff_datetime >= pickup_datetime`, `fare_amount >= 0`, `trip_distance < 500`; `dbt_utils.recency`; row-count reconciliation from source to fact.
- **Inconsistent casting:** green uses `safe_cast`, yellow uses `cast`. Pick one. The project's `safe_cast` also shadows dbt's built-in `dbt.safe_cast`, so the built-in could be used instead.
- **Stale comment:** `int_trips.sql` references `analyses/trips_data_quality.sql`, which doesn't exist.
- `snapshots/` and `tests/` are empty: there's no SCD2 and there are no singular tests yet. Good material for the "what would you add?" question.

---

## 4. Rapid-fire dbt Q&A (answer each out loud in 30–60 s)

**Fundamentals**
1. **`ref()` vs. `source()`?** `ref` points at another dbt model or seed. It builds the DAG, resolves the right schema per environment, and sets build order. `source` points at raw tables not built by dbt, and adds freshness checks and lineage.
2. **Materializations?** `view` (cheap, always fresh, slow for consumers), `table` (fast reads, full rebuild), `incremental` (only processes new or changed rows), `ephemeral` (inlined as a CTE, not built in the warehouse, can't be queried directly), `materialized_view` (the warehouse keeps it fresh). Choose by data size, freshness needs and query patterns.
3. **Incremental strategies?** `append` (inserts only, can create duplicates), `merge` (upsert on `unique_key`), `delete+insert`, `insert_overwrite` (replaces partitions; BigQuery/Spark), `microbatch` (dbt 1.9+; processes time batches by `event_time` with built-in lookback and per-batch retry and backfill).
4. **`is_incremental()` is true when?** When the model already exists, isn't a view, isn't run with `--full-refresh`, and is materialized as incremental.
5. **`{{ this }}`?** The model's own existing relation in the warehouse. Used in incremental filters.
6. **`on_schema_change` options?** `ignore` (default), `fail`, `append_new_columns`, `sync_all_columns` (adds and removes columns).
7. **What are seeds for, and when *not* to use them?** Small, static, version-controlled lookup data (codes, mappings). Not for large or frequently changing data, and not for anything containing PII.
8. **Macros vs. packages?** Macros are reusable Jinja functions in your project. Packages are other people's macros, models and tests, installed with `dbt deps`.
9. **Jinja is evaluated when?** At compile time, before any SQL runs. `run_query` lets a macro query the warehouse during compilation, e.g. to build a pivot from distinct values.

**Testing and quality**
10. **Generic vs. singular tests?** Generic tests are parameterised and declared in YAML (`unique`, `not_null`, `accepted_values`, `relationships`, or your own `{% test %}`). Singular tests are a single SQL file in `tests/` that fails if it returns any rows.
11. **Unit tests (dbt 1.8+)?** You give mock input rows and the expected output for a model and test its *logic*, not the data. A good fit for the dedup and surrogate-key logic in `int_trips`.
12. **Test severity?** `severity: warn|error`, plus `warn_if`/`error_if` thresholds such as `">10"`. With `store_failures: true`, failing rows are written to a table for debugging.
13. **Contracts vs. tests?** A contract checks the *shape* (column names and types) before the model is built, so a bad build never lands. Tests check *data values* after the build. Contracts plus `versions` are how you change a public model without breaking its consumers.
14. **`dbt build` vs. `run` + `test`?** `build` runs seeds, snapshots, models and tests in DAG order. If a test fails, everything downstream of it is skipped, so bad data doesn't propagate.

**Operations and scale (lead territory)**
15. **Slim CI?** `dbt build --select state:modified+ --defer --state prod-artifacts/` builds only the changed models and their children. `--defer` reads unchanged parents from prod instead of rebuilding them. It's fast and cheap on every PR.
16. **Node selection syntax?** `+model` (model and everything upstream), `model+` (model and everything downstream), `tag:daily`, `path:models/marts`, `--exclude`, `@model` (model, its descendants, and their parents), `result:error`, `source_status:fresher+`.
17. **Environments?** Each target has its own schema (`dbt_<user>` for dev, `prod`). `generate_schema_name` decides how custom schemas are named. Dev builds on sampled data (`target.name == 'dev'`).
18. **Snapshots / SCD2?** `strategy: timestamp` (uses an `updated_at` column) or `check` (compares the listed columns). dbt adds `dbt_valid_from`, `dbt_valid_to`, `dbt_scd_id`. In the taxi project, a snapshot of `taxi_zone_lookup` would track zone renames over time.
19. **dbt Core vs. Cloud?** Core is the open-source CLI: you run it yourself with an orchestrator (Airflow, Kestra, Dagster). Cloud adds an IDE, job scheduler, hosted docs, CI integration, the Semantic Layer, and Explorer. Choose by team size, budget, and existing orchestration.
20. **Governance at scale?** `groups` with `access: private | protected | public`, model `versions`, contracts, and dbt Mesh (cross-project `ref`) to split a 2,000-model monolith by domain. `exposures` document dashboards that depend on models, which shows downstream impact.
21. **Docs and lineage?** `dbt docs generate` produces `manifest.json` and `catalog.json`. Persist descriptions to the warehouse with `persist_docs`. Use `meta` for owner and PII tags.
22. **Semantic layer / metrics?** Defining a metric once (MetricFlow) stops "revenue" from meaning three different things in three dashboards.
23. **Performance debugging?** `target/run_results.json` shows execution time per model. Check the compiled SQL in `target/compiled/`. Look at bytes scanned in the warehouse. Then decide what to materialize or make incremental, and what to partition or cluster.
24. **Orchestrating dbt?** Kestra or Airflow runs `dbt build` after the ingestion tasks finish, using `source_status:fresher+` to build only what has new data, with alerts on failures and on `run_results`.

**Analytics engineering concepts**
25. **What is analytics engineering?** The role between data engineering and data analysis: applying software practices (version control, tests, CI, modularity, docs) to transformation code, so analysts get trustworthy, well-modelled data.
26. **Kimball star schema vs. One Big Table in a columnar warehouse?** A star schema gives reuse, conformed dimensions and easier SCD handling. Wide denormalised tables give fewer joins and simpler BI. In practice: build a star in marts, and add wide "reporting" tables on top for dashboards. This project does exactly that with `fct_monthly_zone_revenue`.
27. **Fact grain?** "One row per trip" for `fct_trips`, and "one row per zone × month × service type" for the revenue mart. Always state the grain first. Mixing grains is the most common modelling bug.

---

## 5. Whiteboard exercises (do them, then compare with §3)

1. **Rewrite `fct_trips` for BigQuery** with `insert_overwrite`, daily partitions and a 3-day lookback. Explain why it's idempotent.
2. **Write a unit test** (YAML) for `int_trips` with two rows that share the key columns but have different dropoff times. Expect one output row (the earliest dropoff).
3. **Write a singular test** `tests/assert_dropoff_after_pickup.sql` that returns trips where `dropoff_datetime < pickup_datetime`.
4. **Add a snapshot** for zones (`strategy: check`, `check_cols: [borough, zone, service_zone]`). Then explain how `fct_trips` would join to the zone version valid at `pickup_datetime`.
5. **Design question:** "Analysts complain the monthly revenue dashboard and the finance report disagree by 2%." Walk through the investigation: definitions (`fare_amount` vs. `total_amount`; refunds, negative amounts), grain, dedup differences, late data (issue #1), time zones, then the fix (one metric definition plus a reconciliation test).
6. **Scale question:** "The project grows to 800 models and 15 contributors, and CI takes 40 minutes." Answer: slim CI with `--defer`, tag-based job splitting, incremental and microbatch models, domain ownership with groups and access, dbt Mesh, model contracts on public models, and a style guide enforced with sqlfluff and pre-commit hooks.

---

## 6. Answer template for any dbt question

**What it is → how this project uses it (name the file) → the trade-off or failure mode → what you'd do as a lead.**

Example: *"Incremental models only process new rows. Our `fct_trips` merges on a surrogate `trip_id`. The risk is late-arriving data, because our filter is a strict `>` on max pickup time. I'd add a 3-day lookback, partition by day, and on BigQuery probably switch to `insert_overwrite`, which is cheaper and idempotent. Then I'd schedule a monthly full refresh to reconcile."*
