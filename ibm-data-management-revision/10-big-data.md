# 10. Big data

> This section covers big data as a concept (the V's, storage and processing choices) and the Hadoop-era building blocks still found in interviews and legacy estates: Hadoop, HDFS, MapReduce, Avro and Ranger. Even where Spark, object storage and lakehouses have replaced them, the ideas carry over directly: block replication, data locality, map/shuffle/reduce, schema evolution and centralised fine-grained authorisation. Interviewers ask about them to check that you understand *why* modern tools work the way they do.

## Big data
*Source: [IBM — What is Big Data?](https://www.ibm.com/think/topics/big-data)*

**In one line:** Big data means datasets so large, fast-arriving or varied that traditional single-server data management can't store or process them well.

**Key points**
- The five V's: **volume** (amount), **velocity** (speed of arrival), **variety** (structured, semi-structured, unstructured), **veracity** (accuracy and trustworthiness) and **value** (real business benefit).
- Traditional data is mostly structured and relational, small enough for SQL and classical statistics. Big data needs distributed processing plus ML, data mining and visualisation.
- How it evolved: Hadoop brought distributed storage and parallel processing on clusters → Spark added in-memory computing → data lakes added cheap schema-on-read storage → cloud added elastic, pay-per-use capacity.
- Management stages: collection (Kafka for streams, NiFi for flow automation; capture metadata; validate and deduplicate early) → storage → analytics.
- Storage choices: **data lake** (cheap, raw, any format), **data warehouse** (cleaned, schema-enforced, fast BI), **lakehouse** (lake flexibility with warehouse structure). Many organisations use more than one.
- Processing tools: Hadoop (cheap batch at scale), Spark (fast in-memory, streaming, ML), NoSQL (horizontally scalable document, key-value and graph stores).
- Challenges: quality, scalability, privacy and security (GDPR/HIPAA), integration complexity, scarce skills. Big data is also the fuel for deep learning and LLMs.

**Compare / don't confuse with:** Lake vs. warehouse vs. lakehouse (raw and cheap vs. curated and fast vs. both, via open table formats).

**Interview angle**
- *Q: How do you decide a problem is "big data" and needs distributed tools?* — When it no longer fits on one well-tuned machine within the time or cost budget, for memory, I/O or concurrency reasons. Often better algorithms, partitioning, columnar formats or a single bigger node are enough, and they are simpler than a cluster.

**Your experience hook:** In the Splink entity-resolution project, blocking cut 14.7B candidate pairs to 77M. That shows the main lesson of big data: reduce the work algorithmically before adding hardware.

**Beyond IBM:** In today's stack, "big data" usually means object storage + Parquet + an open table format (Iceberg/Delta/Hudi), queried by Spark, Trino or a cloud warehouse.

---

## Apache Hadoop
*Source: [IBM — What Is Hadoop?](https://www.ibm.com/think/topics/hadoop)*

**In one line:** Hadoop is an open-source framework for storing and processing very large datasets across clusters of cheap commodity machines.

**Key points**
- It started from Doug Cutting's work (Nutch, then Yahoo) to overcome Nutch's scaling limits. It stores structured, semi-structured and unstructured data without format requirements.
- Core modules: **Hadoop Common** (shared libraries), **HDFS** (distributed file system; blocks replicated, 3 copies by default; NameNode + DataNodes), **YARN** (job scheduling and cluster resource management), **MapReduce** (parallel batch processing on YARN) and **Ozone** (distributed object store).
- Ecosystem: Hive (SQL-like warehouse), HBase (wide-column NoSQL on HDFS), Pig (data-flow language), Sqoop (RDBMS ↔ Hadoop transfer), Flume (stream logs into HDFS), Oozie (workflow scheduler), ZooKeeper (coordination), Ambari (cluster management), Avro (serialisation), Tez (faster DAG engine replacing MapReduce), Mahout/Submarine (ML), Cassandra, Chukwa.
- Written in Java. Hadoop Streaming lets any executable (e.g. Python) act as mapper or reducer.
- **Spark vs. Hadoop MapReduce:** Spark keeps data in memory between steps, so it is much faster for iterative and interactive work, and it supports streaming and graph processing. Hadoop MapReduce is disk-based batch.
- Use cases: richer decisions from semi-structured data, self-service data science, and offloading "cold" data from expensive systems.

**Compare / don't confuse with:** Hadoop (the whole platform: storage + resource manager + engines) vs. Spark (a processing engine that can run on YARN, Kubernetes or standalone, reading HDFS or object storage).

**Interview angle**
- *Q: What replaced the pieces of Hadoop in modern stacks?* — HDFS → cloud object storage (S3/GCS/ADLS); MapReduce → Spark/Flink/SQL engines; YARN → Kubernetes or managed services; Hive tables → Iceberg/Delta with a catalog; Oozie → Airflow and similar orchestrators.

**Beyond IBM:** The Hadoop ideas still in use today are moving compute to the data, assuming hardware will fail, and partitioned files plus a metastore. The tools changed; the ideas stayed.

---

## HDFS (Hadoop Distributed File System)
*Source: [IBM — What is Hadoop Distributed File System (HDFS)?](https://www.ibm.com/think/topics/hdfs)*

**In one line:** HDFS is Hadoop's fault-tolerant distributed file system. It splits large files into replicated blocks spread across many commodity servers.

**Key points**
- **Architecture:** one **NameNode** (holds the namespace and metadata: files, permissions, block locations) and many **DataNodes** (store blocks as local files and serve reads and writes). User data never passes through the NameNode.
- Files are split into large blocks (128 MB by default in Hadoop 2.x) to reduce seek time and metadata. Each block is replicated (3 copies by default, configurable per file), with at least one copy on a different rack (**rack awareness**).
- DataNodes send **heartbeats** and **block reports** to the NameNode. A DataNode that stops sending heartbeats is treated as dead and its blocks are re-replicated elsewhere.
- Files are **write-once, single-writer**. The design favours high-throughput streaming reads for batch jobs, not low-latency interactive access.
- **Data locality:** computation runs on the nodes that hold the data, which cuts network traffic.
- Benefits: fault tolerance, very large datasets (GB to TB per file, PB per cluster), portability, horizontal scale to thousands of nodes, snapshots.
- HBase sits *on top of* HDFS and adds random, real-time reads and writes. It doesn't replace HDFS.

**Compare / don't confuse with:** HDFS (file system; batch, append-only) vs. HBase (database on HDFS; random access by row key). HDFS vs. object storage (S3: no NameNode, scales metadata differently, no data locality, cheaper and elastic).

**Interview angle**
- *Q: What is the "small files problem" in HDFS?* — The NameNode keeps metadata for every file and block in memory, so millions of tiny files exhaust it, and each small file also creates task overhead. Fix it by compacting into larger files (or container formats such as Avro/Parquet/SequenceFile).
- *Q: What happens when a DataNode dies?* — Its heartbeats stop, the NameNode marks it dead, finds blocks that now have too few replicas, and schedules new copies from the surviving replicas.

**Beyond IBM:** Older clusters had the NameNode as a single point of failure. HA (active/standby NameNodes with shared edit logs) fixed that, and Hadoop 3 added erasure coding to cut the 3× storage cost for cold data.

---

## MapReduce
*Source: [IBM — What is MapReduce?](https://www.ibm.com/think/topics/mapreduce)*

**In one line:** MapReduce is a programming model that splits a big job into parallel *map* tasks that emit key/value pairs and *reduce* tasks that aggregate all values for each key.

**Key points**
- Flow: **input** (usually HDFS) → **split** into blocks across mappers → **map** (records → key/value pairs) → **shuffle and sort** (all pairs with the same key go to the same reducer) → **reduce** (aggregate per key) → **result** written to HDFS.
- Classic example: find the maximum temperature per city. Mappers emit `<city, temp>`, the shuffle groups by city, and each reducer outputs the max.
- The framework handles scheduling, data transfer, fault tolerance and redundancy, so programmers focus on the logic.
- Benefits: scales to petabytes, parallel processing with data locality, works from many languages, open source.
- Use cases: ETL, log analysis, word counts and text mining, tabulation, some ML (k-means, collaborative filtering), image processing.
- Created at Google (Dean & Ghemawat, 2004). Hadoop MapReduce became the most popular implementation.
- Drawbacks: intermediate data is written to disk, not kept in memory, and iterative algorithms need chained jobs. That is why Spark has largely replaced it.

**Compare / don't confuse with:** MapReduce (disk between every stage; a fixed two-stage model) vs. Spark (in-memory DAG of many stages; lazy evaluation; caching).

**Interview angle**
- *Q: Why is the shuffle the expensive part, and how do you reduce it?* — It moves data across the network and sorts it by key. Reduce it by filtering and projecting early, using a *combiner* (local pre-aggregation on the map side), choosing good partition keys to avoid skew, and broadcasting small tables instead of shuffling them for joins.

**Your experience hook:** Splink's blocking rules partition records by key so only pairs within a block are compared, the same "group by key, then work per group" idea as map/shuffle/reduce. MongoDB's `$group` stage does a reduce-by-key inside an aggregation pipeline.

**Beyond IBM:** The same rules apply to Spark wide transformations (`groupBy`, `join`): stages break at shuffles, so most Spark tuning is shuffle and skew management.

---

## Apache Avro
*Source: [IBM — What Is Apache Avro?](https://www.ibm.com/think/topics/avro)*

**In one line:** Avro is a language-neutral data serialisation format: compact binary data described by a JSON schema, with strong support for schema evolution.

**Key points**
- Avro began in the Hadoop ecosystem for data serialisation and exchange between programs written in any language (Java, Python, Ruby, C, C++ and more).
- The schema is written in JSON (human-readable); the data is binary (compact and fast). Avro files carry the schema with the data.
- Files contain sync markers, so large files can be split for parallel MapReduce processing.
- No code generation is required (it is optional), which suits dynamic and scripting languages.
- **Schema evolution:** handles added, removed and changed fields, so old readers can read new data and new readers can read old data.

**Compare / don't confuse with:** Avro (row-oriented; good for messages, streaming and write-heavy landing data) vs. Parquet/ORC (columnar; good for analytical scans). Avro vs. Protobuf: both binary and schema-based; Avro resolves fields by name using the writer's and reader's schemas, while Protobuf uses numbered field tags.

**Interview angle**
- *Q: How does Avro schema evolution actually work, and what makes a change safe?* — The reader resolves data using both the writer's schema and its own, matching fields by name. Adding a field with a default and removing a field that had a default are compatible. Renaming without an alias or changing to an incompatible type is breaking. A schema registry enforces backward, forward or full compatibility on each new version.

**Beyond IBM:** In Kafka, messages normally carry a small schema ID that points to a schema registry rather than the full schema, which keeps messages small and makes the compatibility check part of the data contract.

---

## Apache Ranger
*Source: [IBM — What Is Apache Ranger?](https://www.ibm.com/think/topics/apache-ranger)*

**In one line:** Apache Ranger is an open-source framework for centrally defining, enforcing and auditing fine-grained access policies across big data platforms.

**Key points**
- Centralised security administration: an admin console where you define policies once, applied across clusters.
- Fine-grained policies at file, folder, database, table and column level, assigned to users, groups or roles.
- **Plugin architecture:** lightweight plugins inside each service (originally Hadoop components; later engines such as Presto/Trino) check requests against Ranger policies, which extended Ranger beyond Hadoop.
- **Centralised auditing** of user access and admin actions, stored in audit stores such as Elasticsearch or Solr.
- IBM presents it as part of an "open data lake analytics" stack (object storage + a SQL engine such as Presto + Ranger), so SQL on the lake still gets warehouse-style access control.

**Compare / don't confuse with:** Authentication (who you are: Kerberos, LDAP/AD, SSO) vs. authorisation (what you may do: Ranger policies). Ranger (policy and enforcement) vs. Apache Atlas (metadata and classification). They are often combined so Atlas tags such as "PII" drive Ranger policies.

**Interview angle**
- *Q: How would you restrict analysts from seeing raw PII in a data lake while still letting them query it?* — Classify PII columns (tags), then apply column masking or hashing and row-level filters by role at the query engine (e.g. Ranger policies on Hive/Trino), audit every access, and give the small set of authorised roles an unmasked view.

**Your experience hook:** PII pseudonymisation and access control on the multi-tenant KYC/KYB platform are the same problem Ranger solves centrally, applied at the application layer.

**Beyond IBM:** Ranger also supports tag-based policies, row-level filtering and dynamic column masking. These features matter more than the Ahana/Presto material the IBM page focuses on.

---

## Self-test

1. Name and define the five V's of big data.
<details><summary>Answer</summary>Volume (amount of data), velocity (speed of arrival), variety (structured, semi-structured and unstructured formats), veracity (accuracy and reliability) and value (business benefit gained).</details>

2. Data lake vs. data warehouse vs. lakehouse?
<details><summary>Answer</summary>Lake: cheap storage of raw data in any format, schema-on-read. Warehouse: cleaned, schema-enforced data optimised for fast BI queries. Lakehouse: warehouse-style tables, ACID and governance on top of lake storage, usually through open table formats.</details>

3. What are Hadoop's core modules and what does each do?
<details><summary>Answer</summary>Hadoop Common (shared libraries), HDFS (distributed replicated storage), YARN (resource management and job scheduling) and MapReduce (parallel batch processing). IBM also lists Ozone (object store).</details>

4. What do the NameNode and DataNodes do?
<details><summary>Answer</summary>The NameNode holds the file system namespace and metadata (files, permissions, block locations) and coordinates access. DataNodes store the blocks, serve reads and writes, and report heartbeats and block lists to the NameNode.</details>

5. Why are HDFS blocks large and replicated with rack awareness?
<details><summary>Answer</summary>Large blocks (128 MB default) reduce seek time and NameNode metadata. Replication (3× by default) with at least one copy on another rack survives node and whole-rack failures and lets reads use bandwidth from several racks.</details>

6. Walk through the MapReduce phases.
<details><summary>Answer</summary>Input → split → map (emit key/value pairs) → shuffle and sort (group by key to reducers) → reduce (aggregate per key) → write the output to HDFS.</details>

7. Why did Spark largely replace MapReduce?
<details><summary>Answer</summary>MapReduce writes intermediate results to disk and needs chained jobs for iterative logic. Spark keeps data in memory across a DAG of stages, which makes it much faster for iterative ML and interactive queries, and it also supports streaming and graph workloads.</details>

8. What is a combiner, and why use it?
<details><summary>Answer</summary>A map-side "mini reducer" that pre-aggregates values per key before the shuffle, cutting the amount of data sent over the network. It is only safe for operations like sum, count and max, where partial results can be combined.</details>

9. Which Avro schema changes are safe in both directions (fully compatible)?
<details><summary>Answer</summary>Adding a field *with a default* and removing a field that had a default. (For backward compatibility alone, where new readers read old data, any field can be removed and new fields need defaults.) Renaming without an alias or changing to an incompatible type breaks readers. Avro resolves fields by name using the writer's and reader's schemas.</details>

10. What does Apache Ranger provide, and how is it different from Kerberos?
<details><summary>Answer</summary>Ranger provides centralised, fine-grained authorisation policies (down to column level, with masking and row filters) enforced by plugins in each service, plus centralised auditing. Kerberos handles authentication (proving identity), not what an identity is allowed to access.</details>
