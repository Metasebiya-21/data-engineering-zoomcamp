# Data Management Study Revision

A concept-by-concept revision of IBM's [2026 Guide to Data Management](https://www.ibm.com/think/topics/data-management-guide): **124 concepts in 13 chapters**, rewritten in plain English for interview preparation.

Every concept follows the same layout:
- **In one line:** the definition
- **Key points:** the main ideas from the IBM article
- **Compare / don't confuse with:** the natural contrast, where one exists
- **Interview angle:** a likely question with a short answer
- **Your experience hook:** a link to my own work, where one is real
- **Beyond IBM:** the practitioner's view where IBM is vague, salesy or out of date

Each chapter ends with a **self-test** with hidden answers.

> The text is paraphrased, not copied, and IBM product pitches (watsonx and similar) are left out. Each entry links to its IBM source article.

---

## If you only have a few hours (priority order for a data engineering interview)

| Priority | Read | Why |
|---|---|---|
| 1 | [08 Data integration](08-data-integration.md): **CDC**, ETL vs. ELT, data contracts | Your core experience. The CDC entry covers resume positions, deletes, ordering, snapshot handoff |
| 2 | [03 Databases](03-databases.md): PostgreSQL, MongoDB, Redis, CAP, OLTP, query optimization | SQL and databases are in every job post |
| 3 | [04 Data platforms](04-data-platforms.md): warehouse, lake, **lakehouse**, Iceberg/Delta, MDM | Warehouse vs. lakehouse comparison tables, how table formats work |
| 4 | [12 Data quality](12-data-quality.md): dimensions → concrete checks, observability, stale data | "How do you know the data is right?" |
| 5 | [06 Data engineering](06-data-engineering.md): pipelines, DAGs, reconciliation, validation, orchestration | Idempotency, backfills, retries/DLQs, Airflow vs. Dagster vs. Kestra |
| 6 | [09 Data processing](09-data-processing.md) and [07 Data transfer](07-data-transfer.md): OLAP, stream processing, streaming platforms, latency | Batch vs. streaming trade-offs |
| 7 | [05 Data architecture](05-data-architecture.md): fabric vs. mesh, data modeling | Design and lead-level conversations |
| 8 | [13 Data governance](13-data-governance.md), [11 Enterprise data management](11-enterprise-data-management.md), [01 Introduction](01-introduction.md), [02 Data](02-data.md), [10 Big data](10-big-data.md) | Vocabulary and context |

---

## Chapters

| # | Chapter | Concepts |
|---|---|---|
| 01 | [Introduction](01-introduction.md) | Data management · Modern data stack · AI-ready data · AI data management · DataOps · DataOps framework · DataOps architecture |
| 02 | [Data](02-data.md) | Data · Structured vs. unstructured data · Data in motion · Unified data · Metadata management · Dataset |
| 03 | [Databases](03-databases.md) | Database · Relational databases · SQL · Primary key · NoSQL databases · OLTP · Vector database · Vector databases for RAG · DBaaS · CAP theorem · Database schema · MongoDB · PostgreSQL · Redis · Elasticsearch · OpenSearch · etcd · Cassandra · HBase · CouchDB · Presto · Query optimization · ER diagram |
| 04 | [Data platforms](04-data-platforms.md) | Data platform · Data warehouse · Data mart · Data lake · Delta Lake · Apache Iceberg · AWS Lake Formation · Data lakehouse · Feature store · Lake vs. warehouse vs. mart · Master data management |
| 05 | [Data architecture](05-data-architecture.md) | Data architecture · Data fabric · Data mesh · Data modeling |
| 06 | [Data engineering](06-data-engineering.md) | Data engineering · Agentic AI data engineering · Data acquisition · Data pipeline · AI data pipeline · Pipeline automation · 10 steps to automate pipelines · DAG · Data cleaning · Data reconciliation · Data validation · Data enrichment · Data wrangling · Data virtualization · Data automation · Data orchestration |
| 07 | [Data transfer](07-data-transfer.md) | File transfer · Data migration · Data movement · Streaming data · Data streaming platform · Data latency |
| 08 | [Data integration](08-data-integration.md) | Data integration · ETL · ELT · Change data capture · Real-time integration · Cloud data integration · AI data integration · Data interoperability · Data contract · Integration techniques and methods |
| 09 | [Data processing](09-data-processing.md) | Data processing · Data retrieval · Data ingestion · OLAP · Stream processing · Unstructured data processing · Real-time data streaming · Complex event processing · Streaming analytics |
| 10 | [Big data](10-big-data.md) | Big data · Hadoop · HDFS · MapReduce · Avro · Ranger |
| 11 | [Enterprise data management](11-enterprise-data-management.md) | EDM · Chief data officer · Data democratization · Data literacy · Data marketplace · Data optimization · Data modernization · Data reduction · Data silos · Data strategy · Data SLA · Data fragmentation |
| 12 | [Data quality](12-data-quality.md) | Data quality · Quality management · Quality dimensions · Accuracy · Integrity · Reliability · Observability · Bad data · Stale data · Dark data · Dirty data · AI data quality |
| 13 | [Data governance](13-data-governance.md) | Data governance · Data stewardship |

---

## Where IBM gets it wrong or oversimplifies (the "Beyond IBM" corrections)
These come up in interviews because they separate surface knowledge from real experience:
- **NoSQL ≠ "always availability over consistency."** MongoDB and HBase are usually described as CP, and Cassandra's consistency is tunable (R + W > RF).
- **"Consistency" means different things in ACID and in CAP:** constraints and valid state vs. every node seeing the same latest value.
- **IBM calls PostgreSQL a NoSQL database** in its Redis article. It's relational.
- **Time travel ≠ rollback.** Querying an old table version is different from restoring it.
- **A GDPR delete on a lakehouse** isn't physically complete until old snapshots or files are removed (VACUUM, expire_snapshots).
- **Kafka is a durable log, not a stream processor.** Processing happens in Flink, Spark or Kafka Streams.
- **Streaming isn't "less reliable" than batch.** Reliability depends on delivery guarantees (at-least-once + idempotent sinks, exactly-once).
- **Data fabric as "the next stage after the lakehouse"** is IBM's product positioning, not industry consensus.
- **PrestoSQL was renamed Trino** in 2020.
- **Licensing has moved on:** Elasticsearch added AGPL (2024). Redis changed its licence (2024), which led to the Valkey fork, and Redis 8 added AGPL (2025).
- **SFTP isn't an "enhancement of FTP".** It's a separate protocol over SSH. FTPS is FTP over TLS.
