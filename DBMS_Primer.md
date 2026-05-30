# DBMS Mastery Primer: From College Fundamentals to FAANG, Data-Engineering, AI-Infra, and Quant Interviews

**TL;DR**
- Build mastery in three layers and skip none: (1) single-node DB internals (storage, indexing, transactions, recovery, query optimization) anchored by CMU 15-445 + Petrov's *Database Internals*; (2) distributed data systems anchored by Kleppmann's *Designing Data-Intensive Applications* + the Red Book + canonical papers (Dynamo, Spanner, Snowflake, Bigtable, GFS, Raft); (3) vertical specialization (FAANG system design / lakehouse / vector + RAG / time-series + ACID-strict finance). Interviewers grill the boundary between these layers, so reasoning trumps memorization.
- The single biggest interview signal is your ability to make *trade-off* arguments under questioning ("why B+Tree not LSM here?", "why snapshot isolation not serializable?", "why hash partition not range?"). Memorizing definitions fails; reasoning from first principles + naming the canonical paper wins.
- What's changed 2023–2026: vector DBs + HNSW are table-stakes at AI labs; the Iceberg/Delta/Hudi format war has effectively collapsed onto Iceberg as the open standard (Databricks reportedly paid nearly $2 B for Tabular, the Iceberg creators, in June 2024 per TechCrunch citing Bloomberg) with Delta UniForm bridging; serverless DBs (Aurora Serverless v2, Neon, PlanetScale, Snowflake) and HTAP (TiDB, SingleStore) are mainstream; Discord's 2022 Cassandra→ScyllaDB migration is now a canonical interview case study; "design a RAG architecture" is a standard system-design prompt at OpenAI/Anthropic-style shops.

---

## PART 1 — THE STUDY GUIDE

### 1. Relational Model, Relational Algebra, Set Theory

**Definition / intuition.** A relation is a set of tuples over a tuple schema. SQL is a *declarative* surface; relational algebra (σ select, π project, ⋈ join, ∪ union, − difference, × cartesian product, ρ rename, γ group/aggregate) is the *operational* algebra the optimizer uses internally. E. F. Codd, "A Relational Model of Data for Large Shared Data Banks," *Communications of the ACM*, Vol. 13, No. 6, June 1970, pp. 377–387, DOI 10.1145/362384.362685, founded the field by decoupling the logical model from the physical layout.

**Why it matters in industry.** Every modern engine — Postgres, MySQL, Spanner, Snowflake, BigQuery, DuckDB — parses SQL into a logical algebra tree, rewrites it (predicate pushdown, join reordering, subquery flattening), then picks a physical plan. If you can read an `EXPLAIN`, you can diagnose 80 % of production performance issues.

**Cross-links.** Set theory + first-order logic (a SQL `WHERE` is a predicate; a `JOIN` is relational composition). Compilers: parser → AST → IR → optimizer → codegen mirrors the DB query pipeline exactly.

**Grill-down.** "Translate `SELECT name FROM emp WHERE dept='ENG'` to relational algebra" → π_name(σ_dept='ENG'(emp)). "Is SQL relationally complete?" Yes for σ, π, ×, ∪, −, but it adds bag semantics, NULLs (three-valued logic), and ordering that pure relational algebra lacks.

**References.** Codd 1970; Garcia-Molina/Ullman/Widom *Database Systems: The Complete Book* ch. 2–5.

### 2. SQL Deep Dive

**Layers.** DDL (`CREATE`, `ALTER`, `DROP`, `TRUNCATE`), DML (`SELECT`, `INSERT`, `UPDATE`, `DELETE`, `MERGE`), DCL (`GRANT`, `REVOKE`), TCL (`BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT`).

**Joins.** Inner, left/right/full outer, cross, semi (`EXISTS`), anti (`NOT EXISTS`), lateral. `LEFT JOIN … WHERE right.col IS NULL` is the canonical anti-join idiom; optimizers may rewrite it.

**Subqueries vs CTEs vs window functions.** A correlated subquery executes per outer row unless decorrelated by the optimizer (Postgres usually does, MySQL historically didn't). CTEs (`WITH`) were optimization fences (always materialized) in Postgres ≤ 11; from Postgres 12 they are inlined unless `MATERIALIZED` is specified. Window functions (`ROW_NUMBER() OVER (PARTITION BY x ORDER BY y)`, `LAG`, `LEAD`, `RANK`, `DENSE_RANK`, `NTILE`, `SUM() OVER (...)`) are the single most under-used SQL feature — they replace self-joins and procedural loops with one declarative pass.

**Query plan literacy.** `EXPLAIN ANALYZE` (Postgres) → seq scan vs index scan vs index-only scan vs bitmap heap scan; nested-loop vs hash vs merge join; sort vs hashagg. Read the *actual rows* vs *estimated rows* gap — a 10×+ gap is the #1 source of bad plans (stale stats / bad cardinality estimate).

**Grill-down.** `WHERE` vs `HAVING`. `UNION` vs `UNION ALL` (the former dedupes). "Top-N per group" → `ROW_NUMBER() OVER (PARTITION BY g ORDER BY m DESC)` then filter `= 1`.

**References.** Markus Winand, *SQL Performance Explained* + **Use The Index, Luke!** (use-the-index-luke.com); Postgres `EXPLAIN` docs; sqlzoo.net; LeetCode SQL 50; HackerRank SQL.

### 3. ER Modeling & Schema Design

Entity, attribute, relationship; cardinality (1:1, 1:N, M:N); weak entities; ISA hierarchies. Map ERD → relations: M:N becomes a junction table; ISA becomes single-table-inheritance + discriminator, class-table-inheritance, or concrete-table-inheritance.

**Industry signal.** A clean ER diagram + sensible PK/FK story + correct M:N modeling separates strong from average candidates immediately on any "design the schema for X" prompt. Bank of America's reported SQL prompt on DataLemur is representative: *"A customer can have multiple loans. Each loan is of a specific type (home, auto, personal, etc.) Each loan transaction includes detail about date, principal, interest, etc. … Write a SQL query that gives the current outstanding of each loan for each customer."*

### 4. Normalization (1NF → BCNF, 4NF, 5NF) and Denormalization

- **1NF**: atomic values, no repeating groups.
- **2NF**: 1NF + no partial dependency on a composite key.
- **3NF**: 2NF + no transitive dependency (every non-key attribute depends on the key, the whole key, and nothing but the key).
- **BCNF**: every functional-dependency determinant is a superkey (stricter than 3NF).
- **4NF**: no nontrivial multi-valued dependencies.
- **5NF (PJ/NF)**: no nontrivial join dependencies.

**Denormalization trade-off.** Normalized schemas minimize update anomalies but require joins on read. OLTP wants 3NF/BCNF for write integrity; OLAP star schemas deliberately denormalize into wide fact + dimension tables for scan-friendly columnar reads. NoSQL document stores often denormalize aggressively to embed read-time aggregates.

**Grill-down.** "Give an example where BCNF and 3NF differ." Classic: a relation with two overlapping candidate keys where a non-prime attribute determines a prime attribute.

### 5. Indexing — The Most-Grilled Topic

| Index | Structure | Best for | Cost |
|---|---|---|---|
| B+Tree | Balanced n-ary tree, leaves linked | Range, equality, ordered scans, prefix lookup | Random-write amplification |
| Hash | Hash table | Equality only | No range; collision/resize |
| LSM-Tree | Memtable + sorted SSTables + compaction | Write-heavy, sequential I/O | Read amplification, compaction overhead |
| Bitmap | One bitmap per distinct value | Low-cardinality columns, OLAP | Bad for high cardinality, costly updates |
| Inverted | Term → posting list | Full-text search | Storage overhead, scoring complexity |
| GiST / SP-GiST | Generalized search tree | Geo, range types, similarity | Postgres abstraction |
| GIN | Generalized inverted | Arrays, JSONB, tsvector | Slow inserts |
| BRIN | Block-range min/max | Huge, naturally clustered tables | Useless if data isn't clustered |
| HNSW | Hierarchical small-world graph | High-dim vector ANN | Memory-heavy |
| IVF (+PQ / SQ) | k-means buckets, optionally quantized | Vector ANN at scale | Recall drop with quantization |

**Composite indexes.** `(a,b,c)` answers `WHERE a=?`, `WHERE a=? AND b=?`, etc., plus range scans on a prefix. It does *not* answer `WHERE b=?` alone (leftmost-prefix rule).

**Covering / index-only scans.** Include all referenced columns in the index so the engine never touches the heap. Postgres: `CREATE INDEX … INCLUDE (col)`.

**Partial indexes.** `CREATE INDEX … WHERE deleted_at IS NULL` — index only the hot subset.

**Cross-links.** Data structures (B-trees, skip lists, tries, HNSW = graph + skip-list); OS (page cache, mmap vs explicit buffer pool, sequential vs random I/O).

**Grill-down.** "Why B+Tree not B-Tree?" Leaves linked → range scans without re-traversal; values only at leaves → uniform tree height and predictable I/O. "Why LSM for Cassandra/RocksDB/ScyllaDB?" Sequential writes on SSD; defers cost to background compaction; trades read amp for write amp.

**References.** Petrov *Database Internals* Part I; **Use The Index, Luke!**; Postgres docs on index types; Pinecone "Hierarchical Navigable Small Worlds (HNSW)" + "Nearest Neighbor Indexes for Similarity Search."

### 6. Storage Engines, Page Layout, MVCC, WAL, Buffer Pool

**Row vs columnar.** Row stores (Postgres, MySQL InnoDB, Oracle) co-locate a tuple's columns → cheap point lookups, expensive scans of one column over many rows. Column stores (Snowflake, Redshift, ClickHouse, BigQuery, DuckDB, Parquet/ORC files) co-locate a column across rows → cheap scans + heavy compression (RLE, dictionary, bit-packing, delta) + vectorized execution.

**Page layout.** Fixed-size pages (4–16 KB) with a slotted-page header storing tuple offsets; tuples grow inward from the end while the slot array grows from the start. Snowflake uses PAX / "hybrid columnar" — file-level horizontal partitions, column-major within each.

**MVCC.** Each row carries `(xmin, xmax)` (Postgres) or a version chain (InnoDB undo log). Readers never block writers; writers never block readers. Price: dead-tuple bloat → `VACUUM`/compaction.

**WAL.** Write-ahead logging: log record durable on disk *before* the corresponding data page is flushed. Enables crash recovery (redo committed, undo aborted) and physical replication (Postgres streaming replication ships WAL).

**Buffer pool.** In-memory cache of pages with a replacement policy (LRU-K, CLOCK, 2Q). The DB manages it itself rather than trusting the OS page cache because it knows access patterns (e.g., a sequential scan must not evict the hot index root).

**Cross-links.** OS (mmap, O_DIRECT, fsync semantics, write barriers); compilers (vectorized execution = SIMD codegen in modern engines like DuckDB and Photon).

### 7. Query Execution & Optimization

**Pipeline.** Parser → binder/resolver → logical plan (relational algebra tree) → optimizer → physical plan → executor.

**Rule-based vs cost-based.** RBO applies heuristics ("push predicates down, always"). CBO costs alternative plans using statistics (histograms, MCVs, ndistinct) and picks the cheapest. All serious engines are CBO: Postgres, Oracle, SQL Server, Snowflake, BigQuery.

**Statistics & cardinality estimation.** Histograms (equi-width, equi-depth), most-common-value lists, distinct-value estimators (HyperLogLog), correlation stats. Bad estimates → bad plans → 1000× slowdowns.

**Join algorithms.**
- **Nested loop**: O(n·m); competitive only when inner side is tiny or has a fast index.
- **Hash join**: build hash on smaller side, probe with larger; O(n+m) but memory-hungry (spill to disk if needed).
- **Sort-merge join**: sort both sides on the join key, then merge; ideal when inputs already sorted or for huge equi-joins.

**Cross-links.** Compilers (Cascades = Volcano-style top-down optimizer with memoization; used by SQL Server, Cockroach's `optd`, and inspired Calcite). Distributed systems (in Spark/Trino, optimization also minimizes shuffle).

**Grill-down.** "When does the optimizer pick hash over merge?" "How does Postgres estimate `SELECT * FROM t WHERE a=1 AND b=2`?" (Default: multiplies per-column selectivities assuming independence — a famous source of bad plans; fix with `CREATE STATISTICS` extended statistics.)

**References.** Selinger et al. 1979 (System R optimizer); Goetz Graefe, "The Cascades Framework for Query Optimization"; CMU 15-721 Pavlo lectures.

### 8. Transactions, ACID, Isolation Levels, Anomalies

**ACID.** Atomicity, Consistency, Isolation, Durability.

| Level | Dirty read | Non-repeatable read | Phantom | Lost update | Write skew |
|---|---|---|---|---|---|
| Read uncommitted | ✓ | ✓ | ✓ | ✓ | ✓ |
| Read committed | — | ✓ | ✓ | ✓ | ✓ |
| Repeatable read | — | — | ✓ (per ANSI; Postgres/Oracle "RR" = snapshot, no phantoms) | — | ✓ |
| Snapshot isolation | — | — | — | — | ✓ (per Wikipedia: *"Potential inconsistency problems arising from write skew anomalies can be fixed by adding (otherwise unnecessary) updates to the transactions in order to enforce the serializability property."*) |
| Serializable | — | — | — | — | — |

**Implementations.**
- **2PL (two-phase locking)**: acquire all locks before releasing any; SS2PL holds write locks to commit. Used by SQL Server, MySQL repeatable-read (with gap locks).
- **MVCC**: Postgres, Oracle, MySQL InnoDB, Spanner, CockroachDB.
- **SSI (Serializable Snapshot Isolation)**: Postgres' `SERIALIZABLE` since 9.1; detects rw-antidependency cycles at commit and aborts one party. Optimistic.
- **TrueTime / external consistency**: Spanner uses commit-wait against `TT.now().latest` to guarantee external (linearizable) consistency globally. Per Google Cloud's official docs: *"Because Spanner uses multi-version concurrency control (MVCC), the ordering guarantee on timestamps enables clients of Spanner to perform consistent reads across an entire database (even across multiple Cloud regions) without blocking writes."*

**Anomalies.** Dirty read; non-repeatable read; phantom (range insert visible to a re-query); lost update (two RMW transactions clobber each other); write skew (two transactions read overlapping sets, write disjoint sets, but the invariant requires considering both — the canonical doctor-on-call example).

**Grill-down.** "What does Postgres `REPEATABLE READ` actually give you?" Snapshot isolation (blocks phantoms in practice but not write skew). "How to prevent write skew without `SERIALIZABLE`?" Materialize the conflict with `SELECT … FOR UPDATE` on a sentinel row, or use SSI, or app-level optimistic locking with a version column.

### 9. Concurrency Control Deep Dive

- **Pessimistic**: locks (shared, exclusive, intent — IS, IX, SIX); deadlock detection (wait-for graph) or prevention (wait-die, wound-wait); 2PL.
- **Optimistic**: validate-at-commit (OCC); good for low-conflict workloads.
- **Timestamp ordering**: Thomas write rule.
- **MVCC**: keeps version chain; readers see consistent snapshot; writers may abort on conflict.

**Cross-links.** OS (mutex, RW-lock, futex, hazard pointers); distributed systems (locks across nodes require a coordinator → 2PC or leases).

### 10. Recovery (ARIES, WAL, Checkpoints)

**ARIES** (Mohan et al., IBM, 1992) — three phases on restart:
1. **Analysis** — scan forward from last checkpoint, rebuild Transaction Table & Dirty Page Table.
2. **Redo** — replay history from earliest `recLSN` (the repeating-history principle: redo everything, even aborted txns).
3. **Undo** — roll back transactions still active at crash, using CLR (Compensation Log Records) to make undo idempotent.

Steal/no-force buffer policy (allow dirty pages out before commit; do not force pages at commit) is the standard, only safe because of WAL.

**Fuzzy checkpoints.** Do not quiesce the system; snapshot the TT/DPT + flush the log buffer.

**Cross-links.** Distributed systems (Raft log = WAL for state machine; Paxos log is the same idea).

**Reference.** Mohan et al. 1992 "ARIES" (Red Book ch. 3); CMU 15-445 lecture 21.

### 11. Distributed Databases

**CAP.** Under a network partition, choose Consistency or Availability. PACELC (Abadi) extends it: Else (no partition), choose Latency or Consistency. Spanner is CP+EC; Dynamo/Cassandra is AP+EL; CockroachDB is CP+EC.

**Consensus.**
- **Paxos** (Lamport 1998): basic Paxos = one decision; Multi-Paxos = a log of decisions with a stable leader.
- **Raft** (Diego Ongaro and John Ousterhout, "In Search of an Understandable Consensus Algorithm," USENIX ATC 2014, Philadelphia, pp. 305–319 — Best Paper Award per raft.github.io): explicit leader election (term + randomized timeout), log replication, safety via "election restriction." Used by etcd, CockroachDB, TiKV, YugabyteDB. Per cockroachlabs.com/docs/stable/architecture/reads-and-writes-overview: *"For a write request, the Raft consensus protocol dictates that a majority of the replicas of the relevant range must agree before the write is committed."*

**Replication topologies.**
- Single-leader (Postgres streaming, MySQL replica) — strong consistency on leader, async lag on followers.
- Multi-leader (Galera, CockroachDB, BDR) — conflict resolution required (LWW, CRDTs).
- Leaderless (Dynamo, Cassandra, Riak) — quorum reads/writes (R+W>N), hinted handoff, read repair, anti-entropy via Merkle trees.

**Sharding/partitioning.**
- Hash partitioning: even distribution; no cross-shard range scans.
- Range partitioning: cheap range scans; hot-spot risk.
- Consistent hashing (Karger et al. 1997): N nodes → O(K/N) keys move on resize.

**2PC / 3PC.** 2PC blocks if coordinator fails after prepare. 3PC adds pre-commit to avoid blocking but assumes synchronous network. In practice, replace with consensus-coordinated commit (Spanner's Paxos Commit; Cockroach's Parallel Commits).

**Saga.** Long-running multi-service "transaction" → sequence of local transactions + compensating actions. Orchestrated (central coordinator) or choreographed (event-driven).

**Cross-links.** Networking (RTT, head-of-line blocking, NTP/PTP for clocks); distributed systems (FLP impossibility, Lamport clocks, vector clocks).

**References.** Lamport "Paxos Made Simple"; Ongaro & Ousterhout 2014 (Raft); Brewer CAP; Abadi PACELC; **Dynamo paper (SOSP 2007)** which enumerated *"Consistent Hashing, Vector clocks with reconciliation during reads, Sloppy Quorum and hinted handoff, Anti-entropy using Merkle trees, Gossip-based membership protocol and failure detection"* as the canonical AP-store recipe.

### 12. NoSQL Families

| Family | Examples | Data model | When to pick |
|---|---|---|---|
| Key-value | DynamoDB, Redis, etcd, Riak | Opaque blob by key | Sessions, caches, simple lookups |
| Document | MongoDB, Couchbase, Firestore | JSON/BSON, secondary indexes | Heterogeneous schema, embedded aggregates |
| Wide-column | Cassandra, ScyllaDB, HBase, Bigtable | Row key → column families | Massive write throughput, time-series, append-heavy |
| Graph | Neo4j, Neptune, JanusGraph, TigerGraph | Nodes + edges + properties | Many-hop traversals, fraud, social, knowledge graphs |
| Search | Elasticsearch, OpenSearch, Solr | Inverted index | Full-text, log analytics |

**Grill-down.** "Why is Cassandra fast at writes?" LSM-tree + append-only + leaderless; writes never block on reads or compaction. "Why did Discord migrate Cassandra → ScyllaDB?" Per Discord's "How Discord Stores Trillions of Messages" (Bo Ingram, March 2023): *"With Cassandra, we struggled with hot partitions. High traffic to a given partition resulted in unbounded concurrency, leading to cascading latency."* Discord reports *"fetching historical messages had a p99 of between 40–125ms on Cassandra, with ScyllaDB having a nice and chill 15ms p99 latency … we're going from running 177 Cassandra nodes to just 72 ScyllaDB nodes."*

### 13. NewSQL

Scale-out SQL with ACID. **Spanner** (globally distributed, TrueTime), **CockroachDB** (Postgres wire, Raft per range), **TiDB** (MySQL wire, TiKV+Raft, Percolator-style txns), **YugabyteDB** (Postgres wire, Raft per tablet), **VoltDB** (deterministic execution), **SingleStore** (HTAP, vectorized).

### 14. OLTP vs OLAP vs HTAP

- **OLTP**: short txns, point reads/writes, row store, 3NF. Postgres/MySQL/Oracle/DynamoDB.
- **OLAP**: long scans, aggregations, columnar, star schema. Snowflake/BigQuery/Redshift/ClickHouse/Druid.
- **HTAP**: one system serving both via dual storage (row + column) — TiDB+TiFlash, SingleStore, SAP HANA.

### 15. Data Warehousing

**Kimball star schema** = central fact table (measures + FKs to dimensions) + denormalized dimensions. **Snowflake schema** = normalized dimensions (saves storage, costs joins). **SCD** (Slowly Changing Dimensions): Type 1 overwrite, Type 2 row-versioning with effective/end dates, Type 3 add column, Type 6 hybrid. Fact-table grain is sacred — define it before you build anything.

### 16. Data Lakes, Lakehouses, Table Formats

**Lakehouse** = warehouse-style ACID + schema + time travel on top of an object-store data lake (S3/ADLS/GCS). The 2026 reality is that the three open table formats have converged on similar features but diverged on strengths:

| Format | Origin | Strength | 2026 status |
|---|---|---|---|
| Apache Iceberg | Netflix | Engine-agnostic open spec; hidden partitioning; partition evolution; Puffin stats | De facto industry standard (AWS S3 Tables, Snowflake, BigQuery, Databricks UniForm all read it) |
| Delta Lake | Databricks | Mature; tight Databricks/Spark integration; deletion vectors; UniForm reads/writes Iceberg | Strong inside Databricks |
| Apache Hudi | Uber | Best for upsert-heavy CDC; record-level index; native incremental queries | Specializes in streaming/CDC; now ships an Iceberg-format mode |

Onehouse's October 2025 comparison captures Hudi's CDC differentiator: *"Iceberg has an incremental read, but it only allows you to read incremental appends, not updates and deletes — which are essential for true change data capture (CDC) and transactional data within a data lakehouse."* The format-war signaled its end with Databricks's reported ~$2 B acquisition of Tabular in June 2024 (TechCrunch citing Bloomberg: *"Databricks reportedly paid nearly $2 billion when it acquired Tabular in June, a startup that was only doing $1 million in annual recurring revenue"*).

### 17. Columnar Formats

- **Parquet**: row-groups → column chunks → pages; dictionary + RLE + delta + zstd/snappy.
- **ORC**: stripe → column → stream; great Hive integration.
- **Avro**: row-oriented; great for streaming (Kafka), schema evolution via Avro schemas.

**Cross-link.** Compilers — Parquet decoders use SIMD; **Arrow** is the in-memory columnar IR shared by Spark, DuckDB, Polars, and Pandas 2.

### 18. Streaming and CDC

- **Kafka**: distributed log; partitioned topics; offsets; consumer groups; exactly-once via idempotent producer + transactional API.
- **Debezium**: parses DB WAL (Postgres logical replication, MySQL binlog, Mongo oplog) → Kafka topic. Standard CDC pattern.
- **Flink**: true streaming, event time, watermarks, exactly-once via state checkpoints (Chandy-Lamport).
- **Spark Structured Streaming**: micro-batch (continuous mode experimental); end-to-end exactly-once with idempotent sinks.

**Grill-down.** "Exactly-once across the boundary — how?" Producer idempotence + transactional writes spanning sink + offset commit, or Flink's two-phase commit sink aligned with checkpoint barriers.

### 19. Vector Databases & Embeddings

**The two dominant ANN index families:**
- **HNSW** (Hierarchical Navigable Small World): multi-layer graph, greedy descent from top to bottom; O(log N) search. Best recall/latency. Memory-hungry — meta-intelligence.tech's analysis (using HNSW M=16 on 1536-dim float32 vectors) reports *"each vector's memory footprint is approximately 6.5 KB (6 KB for the raw vector + approximately 0.5 KB for the adjacency list), and 100 million vectors require approximately 620 GB of memory."* Pinecone's "Hierarchical Navigable Small Worlds (HNSW)" notes: *"HNSW is not the best index in terms of memory utilization. However, if this is important and using another index isn't an option, we can improve it by compressing our vectors using product quantization (PQ). Using PQ will reduce recall and increase search time — but as always, much of ANN is a case of balancing these three factors."*
- **IVF (+PQ / SQ)**: k-means partitions ("Voronoi cells"); query probes nProbe cells. Lower memory with quantization but recall drops.

**Production options 2026.** Pinecone (managed SaaS, proprietary index); Milvus / Zilliz (11+ index types including HNSW, IVF_FLAT, IVF_PQ, IVF_SQ8, DiskANN, GPU; disaggregated compute/storage); Weaviate (HNSW + hybrid BM25); Qdrant (HNSW + named vectors); pgvector (HNSW + IVF inside Postgres; good for ≤ ~5 M vectors before specialized DBs win); FAISS (a library, embedded in your process).

**Hybrid search** (BM25 + dense + RRF fusion) is now standard at production scale; pure semantic search misses citations and rare proper nouns.

**Cross-links.** Data structures (HNSW = skip-list + graph); ML (the embedding model — text-embedding-3, BGE, E5 — dominates retrieval quality far more than the index).

### 20. Graph Databases

- **Neo4j** — labeled property graph, **Cypher** (`MATCH (a:Person)-[:FRIEND]->(b) WHERE a.name='Alice' RETURN b`).
- **Amazon Neptune** — supports property graph (**Gremlin**, traversal-based) + RDF triples (**SPARQL**).
- **TigerGraph / JanusGraph / ArangoDB**.

**When to pick.** Many-hop traversals (>3 joins), variable-depth paths, fraud rings, knowledge graphs. For 1–2 hop relations, plain SQL `JOIN` wins.

### 21. Time-Series Databases

- **InfluxDB** (TSM tree), **TimescaleDB** (Postgres extension; hypertables; compressed chunks), **Prometheus** (pull-based monitoring), **kdb+/q** (column-store + array language; dominant on Wall Street tick data — its `aj` "as-of join" returns "the most recent records prior to the times in the first" table per kx.com docs, which is the canonical "prevailing quote at the time of trade" primitive — O(log n) in kdb+, expensive in a row store).

### 22. Caching

- **Redis** (data-structure server: strings, lists, hashes, sorted sets, streams, HyperLogLog, Bloom via module; pub/sub; Lua scripting; cluster mode).
- **Memcached** (pure LRU KV; simpler, lower memory overhead).

**Patterns.** Cache-aside (lazy load); write-through (cache + DB); write-back/write-behind (writes to cache, async to DB — risky on eviction); read-through.

**Invalidation.** TTL; explicit invalidation on write; versioned keys (`user:v3:1234`); pub/sub fan-out.

**Famous problems.** Thundering herd / dogpile (request coalescing or probabilistic early expiration); cache stampede on cold start (preheat); cache penetration (Bloom for "known to not exist"); cache avalanche (jitter TTLs).

### 23. Search Engines

Lucene-based: **Elasticsearch**, **OpenSearch**. Inverted index: term → posting list (doc IDs + positions); skip lists for AND; BM25 scoring; analyzers (tokenizer + filters); shards + replicas. Hybrid search now adds dense vectors (HNSW since ES 8 / OS 2).

### 24. Cloud Databases

- **Aurora** (MySQL/Postgres compatible; log-as-database; 6-way storage replication across 3 AZs).
- **DynamoDB** (managed Dynamo-style KV; partition + sort key; GSIs/LSIs; on-demand or provisioned; single-digit-ms p99).
- **BigQuery** (Dremel descendant, serverless, separation of compute/storage, columnar Capacitor format).
- **Snowflake**. The SIGMOD 2016 paper of record (Dageville et al., "The Snowflake Elastic Data Warehouse," ACM SIGMOD 2016, pp. 215–226, DOI 10.1145/2882903.2903741) opens: *"we describe the design of Snowflake and its novel multi-cluster, shared-data architecture."* It has three layers (Cloud Services, Virtual Warehouses, Data Storage), immutable hybrid-columnar (PAX) files in S3, MVCC + time travel + zero-copy clones.
- **Databricks SQL / Photon** — vectorized C++ engine on top of Delta/Iceberg.
- **Redshift** (MPP, ParAccel-derived; RA3 decouples compute/storage; Spectrum for S3).

### 25. Performance Tuning

`EXPLAIN ANALYZE`, `pg_stat_statements`, `auto_explain`, `pg_stat_user_indexes`, slow-query logs. Missing index? Add. Bad estimate? `ANALYZE` or extended stats. Bad join order? Rewrite or `pg_hint_plan`. Plan-cache pollution? Parameterize. Too many round-trips? Batch / pipeline.

### 26. Security

- **AuthN**: passwords, certificates, IAM, OIDC, Kerberos.
- **AuthZ**: RBAC (`GRANT SELECT ON t TO role`), row-level security (`CREATE POLICY … USING (tenant_id = current_setting('app.tenant')::int)`), column masking, ABAC.
- **Encryption**: at rest (TDE — Transparent Data Encryption, KMS-managed keys), in transit (TLS), envelope encryption, BYOK.
- **SQL injection**: parameterized queries always; ORMs are mostly safe if you don't string-concat.
- **Auditing**: `pgaudit`, AWS CloudTrail / DB audit log, immutable WORM log for SOX/HIPAA/PCI-DSS.

### 27. Backup, HA, DR

- **Backup**: logical (`pg_dump`), physical (`pg_basebackup`), incremental (WAL archiving + PITR).
- **PITR**: replay WAL to a target timestamp/LSN.
- **HA**: synchronous replica (zero RPO, latency cost), async replica (lower latency, non-zero RPO), automated failover (Patroni, Orchestrator, Aurora endpoint failover, RDS Multi-AZ).
- **DR**: RPO (data loss tolerance) vs RTO (downtime tolerance); cross-region async replica; documented runbook drills.

---

## PART 2 — THE ROADMAP

### Phase 0 — Prerequisites (~2 weeks)
Confirm: solid SQL basics; one language (Python/Java/Go/C++); OS concepts (processes, threads, file I/O, page cache, virtual memory); basic data structures (arrays, hash tables, trees, heaps); basic networking (TCP, RTT, DNS).
**Milestone:** 30 LeetCode Easy SQL problems.

### Phase 1 — Foundations (~6 weeks)
**Topics.** Relational model, SQL deep dive, ER + normalization, transactions/ACID, basic indexing, single-node storage.
**Resources.**
- Book: *Database System Concepts* (Silberschatz/Korth/Sudarshan) ch. 1–14 — or Garcia-Molina/Ullman/Widom *Database Systems: The Complete Book* ch. 1–10.
- Course: **Berkeley CS186** (Joe Hellerstein, free on YouTube + Spring projects).
- Course: **CMU 15-445** lectures 1–10 (Pavlo, Fall 2023+, free; BusTub C++ projects).
- Site: **Use The Index, Luke!** (Markus Winand) — top to bottom.
- Practice: sqlzoo.net, LeetCode SQL 50, HackerRank SQL Advanced, DataLemur company-tagged.
**Hands-on project.** TPC-H subset on local Postgres; write 10 analytical queries, read `EXPLAIN ANALYZE` for each, rewrite/index to halve latency.
**Milestone.** You can sketch a 3NF schema for any reasonable domain, write joins/CTEs/window functions fluently, and explain `EXPLAIN ANALYZE` line by line.

### Phase 2 — Intermediate / Single-Node Internals (~8 weeks)
**Topics.** B+Tree internals, LSM trees, WAL, ARIES recovery, MVCC, isolation-level implementations, query optimization, join algorithms.
**Resources.**
- Book: **Petrov *Database Internals*** Part I (storage engines, B-trees, LSM, txns/recovery).
- Course: **CMU 15-445** lectures 11–26 (concurrency, recovery, optimization).
- Papers: ARIES (Mohan 1992), Selinger 1979 (System R), Cascades (Graefe), Stonebraker "What Goes Around Comes Around."
- Site: Andy Pavlo's *Database of Databases* (dbdb.io) for taxonomy.
**Hands-on project.** Complete the 4 BusTub projects (buffer pool manager, B+Tree, query executors, concurrency control). **Highest-ROI single project in the roadmap.**
**Milestone.** Implement a B+Tree from scratch with leaf-level concurrent traversal; explain ARIES analysis/redo/undo on paper; articulate Postgres MVCC vs InnoDB MVCC vs Oracle MVCC.

### Phase 3 — Distributed Data Systems (~8 weeks)
**Topics.** CAP/PACELC, consensus (Paxos, Raft, EPaxos), replication topologies, partitioning, 2PC vs Saga, consistency models (linearizability, sequential, causal, eventual), CRDTs.
**Resources.**
- Book: **Kleppmann *Designing Data-Intensive Applications*** — cover to cover. Ch. 5 (Replication), 6 (Partitioning), 7 (Transactions), 8 (Trouble with Distributed Systems), 9 (Consistency & Consensus) are the heart of the interview.
- Book: Petrov *Database Internals* Part II.
- Book: **Readings in Database Systems, 5th Ed.** ("Red Book") — Bailis/Hellerstein/Stonebraker, free at redbook.io.
- Papers (must-read): **Dynamo** (DeCandia 2007), **Bigtable** (Chang 2006), **GFS** (Ghemawat 2003), **MapReduce** (Dean 2004), **Spanner** (Corbett 2012 + Corbett 2017 CAP paper), **F1** (Shute 2013), **Snowflake** (Dageville 2016), **Calvin** (Thomson 2012), **Percolator** (Peng 2010), **Raft** (Ongaro & Ousterhout, USENIX ATC 2014).
- Course: **MIT 6.824 / 6.5840** Distributed Systems labs (Raft, sharded KV).
**Hands-on project.** Implement Raft in Go from scratch (MIT lab 2) → sharded fault-tolerant KV (labs 3, 4). Alternatively, contribute a small fix to TiKV / etcd / Cockroach.
**Milestone.** Whiteboard Raft (leader election + log replication + safety) in 15 minutes. Argue why Spanner's TrueTime gives external consistency while Cockroach's HLC-only does not.

### Phase 4 — Modern Data Platforms (~6 weeks)
**Topics.** OLAP/data warehousing, lakehouse + Iceberg/Delta/Hudi, columnar formats, Spark/Flink internals, Kafka + CDC, streaming joins, schema evolution.
**Resources.**
- Book: Kimball *The Data Warehouse Toolkit* (skim — star schema, SCDs, fact-grain).
- Papers: Snowflake (SIGMOD 2016), Dremel/BigQuery (Melnik 2010), Photon (Behm 2022), Apache Arrow, "Lakehouse" (Armbrust CIDR 2021).
- Course: **CMU 15-721** (Advanced DB Systems, Pavlo) — paper-per-lecture, in-memory + columnar.
- Docs: Iceberg spec, Delta protocol, Hudi docs, Snowflake docs.
**Hands-on project.** Ingest a public dataset (NYC Taxi or GH Archive) → land raw in S3/MinIO → bronze/silver/gold pipeline with Spark/dbt on Iceberg → query with Trino/DuckDB → expose hourly aggregates.
**Milestone.** Explain why columnar + vectorized + shared-nothing wins for OLAP; read a Spark physical plan; compare Iceberg vs Delta on schema/partition evolution and concurrent writes.

### Phase 5 — Specialization (~4–8 weeks per vertical)
Pick 1–2 verticals based on target companies — see Part 3.

**Deepening for all verticals:**
- Engineering blogs: Discord (Cassandra→ScyllaDB), Uber (H3, Schemaless), Netflix (EVCache, data mesh), Stripe (Sorbet, ledger), Figma (Postgres sharding 2024), Notion (data lake), Slack.
- Subscribe: Andy Pavlo's DB news; *The Morning Paper* archive (Adrian Colyer); Murat Demirbas "Paper Trail"; @ifesdjeen.

**Milestone.** A working end-to-end project in your target vertical (see Part 3).

---

## PART 3 — THE INTERVIEW DRILL BOOK

### A) FAANG-Style SWE/Backend (Meta, Google, Amazon, Apple, Netflix, Microsoft)

**What they emphasize.** Schema design + sharding/partitioning + indexing trade-offs + caching + replication, embedded in a system-design prompt. They want to see *reasoning*: clarify scale (QPS, data size, read:write ratio), pick storage with a defensible trade-off.

**High-signal questions actually asked** (collated from Glassdoor, LeetCode Discuss, Blind, Hello Interview, Exponent, IGotAnOffer, DesignGurus):

1. Design Twitter / X home timeline (fan-out-on-write vs fan-out-on-read; celebrity problem).
2. Design Instagram (photo storage in S3; metadata sharded by user_id; CDN; feed).
3. Design Uber's driver-location DB (geospatial: geohash vs H3 vs QuadTree — Uber's own H3 blog: *"H3 enables us to analyze geographic information to set dynamic prices and make other decisions on a city-wide level. We use H3 as the grid system for analysis and optimization throughout our marketplaces."*).
4. Design WhatsApp / Messenger (WebSocket; per-conversation log; delivery receipts).
5. Design Dropbox / Google Drive (chunk hashing + content-addressable storage + metadata DB + sync).
6. Design TinyURL (hash collisions; range counters; cache).
7. Design Netflix video playback / recommendations (CDN tiers, manifest, watch history in Cassandra).
8. Design Yelp / Google Maps nearby search (geo-indexes, PostGIS, Elasticsearch geo_point, H3).
9. Design a rate limiter (token bucket vs sliding window; Redis Lua atomic).
10. Design a distributed counter / "likes" service (sharded counters, eventual aggregation).
11. Design a notification system (Kafka fan-out; per-user inbox; mobile push).
12. Design ticket booking / seat reservation (strong consistency; `SELECT FOR UPDATE`; optimistic concurrency).
13. Design a key-value store from scratch (Dynamo-style: consistent hashing, vector clocks, sloppy quorum, hinted handoff, Merkle anti-entropy).
14. Schema for a multi-tenant SaaS (shared schema + tenant_id + RLS vs schema-per-tenant vs DB-per-tenant).
15. Sharding-strategy: "Your DynamoDB partition is hot. Fix it." (Composite key with random suffix; write-sharding; cache; reshape access pattern.)

**How they grill down.**
- Surface: "Which DB?" → "Postgres."
- Push 1: "Why not DynamoDB?" — make the trade-off (schema flex vs scale, txn semantics, cost).
- Push 2: "Write QPS is now 100×. What changes?" — sharding strategy; async write path; CDC to OLAP.
- Push 3: "Make it global. Two writers in different regions race." — last-writer-wins (Cassandra) vs Spanner external consistency (commit-wait) vs CRDT vs single-leader-per-tenant.
- Push 4: "Back this up and run a DR drill." — PITR; cross-region replica; RPO/RTO numbers.

**Sample expert outline — "Design Twitter home timeline":**
- Clarify: 300 M DAU, ~150 M tweets/day; avg 200 followers; celebrities with 100 M+; read:write ≈ 100:1.
- Model: `tweets(tweet_id PK, user_id, text, ts)` sharded by `tweet_id` (Snowflake-style time-ordered ID); `follows(follower, followee)` sharded by `follower`.
- Hybrid fan-out: precompute timelines for normal users (push on write into Redis sorted set per follower, capped at 800); pull-on-read for celebrities at query time and merge.
- Cache: Redis Cluster for hot timelines + L1 process cache. Tweet content cached separately by `tweet_id`.
- Writes via Kafka → fan-out worker → per-follower `ZADD`. At-least-once with idempotent inserts.
- Cross-region: leader per user-region; eventual cross-region replica; accept brief lag.
- Failure modes: celebrity tweet storm (backpressure → switch to pull); Redis eviction (re-materialize from `tweets`).

**Sample — "Design Uber's location DB":**
- 10 M drivers × 0.25 Hz = 2.5 M writes/s.
- Use H3 at resolution 9 (~0.1 km² hexagons): `(driver_id, h3_index, lat, lng, ts)`. Per the Uber H3 blog: *"H3 has functions for directed edges of grid cells which can represent the movement from one cell to another. Directed edges can be stored as 64-bit integers"* (sample H3 index `8a2a1072b59ffff`).
- Hot path: Redis geo / per-hex sorted set (TTL 30 s) keyed by `h3:9:<index>` → set of driver IDs; lookup = `h3_kRing` over 1–2 rings of the rider's cell.
- Cold path: append-only Kafka topic → Cassandra/ScyllaDB partitioned by `(driver_id, day)`.
- Why H3 over geohash: hexagons have uniform neighbor distance; hierarchical 64-bit integer encoding.
- Grill-down: "JFK airport hot-spot" → cap drivers per cell; higher resolution; rate-limit pings.

**Sample — "Ride-share trip schema with consistency":**
- `trips(trip_id PK, rider_id, driver_id, status, fare_cents, started_at, ended_at, version)`; status state machine enforced by `CHECK` constraints + `app_state_transitions` table.
- Postgres `REPEATABLE READ` for normal reads; `SERIALIZABLE` (SSI) or `SELECT FOR UPDATE` for status transitions (avoid write skew between "completed" and "canceled").
- Money: `NUMERIC(12,2)` or integer cents — never `FLOAT`.

**Common traps.**
- Picking "MongoDB" reflexively.
- Forgetting indexes on FK columns (Postgres does *not* auto-index FKs; MySQL does).
- Ignoring read-replica lag → stale data shown to the user who just wrote.
- "Eventual consistency" hand-wave without saying what anomaly the user actually sees.

**What separates strong from average.** Naming the trade-off out loud, citing the canonical paper/system, and quantifying ("p99 50 ms read → Redis; Postgres replica gives p99 ~5–20 ms").

---

### B) Data Engineering Giants (Databricks, Snowflake, Confluent, dbt Labs, Fivetran)

**What they emphasize.** Lakehouse architecture, columnar storage internals, query engines (Catalyst, Photon, Snowflake's optimizer), Spark internals (Catalyst, AQE, Photon, shuffle, broadcast join, skew handling), streaming (Structured Streaming, Flink), schema evolution, Iceberg/Delta protocol details, MPP architecture, file formats.

**High-signal questions actually asked** (collated from Glassdoor, OphyAI's Databricks guide, DataVidhya, AccentFuture, Interview Query, Blind):

1. Walk through Spark's execution model: driver, executors, stages, tasks, narrow vs wide dependencies.
2. Adaptive Query Execution (AQE) — what runtime problems does it solve (skew, partition coalescing, switching join strategies)?
3. Delta Lake transaction log (`_delta_log/`): JSON commits + Parquet checkpoints + protocol versions.
4. Iceberg vs Delta vs Hudi — when each? (Hudi for upsert-heavy CDC; Delta for general-purpose Databricks; Iceberg for engine-agnostic petabyte analytics.)
5. Implement SCD Type 2 with Delta `MERGE INTO`.
6. Z-Order vs Hilbert clustering vs partitioning.
7. Optimize a slow Spark job: small-files problem, skew, shuffle spill, broadcast threshold, AQE.
8. Streaming + state: dedup and watermarking in Structured Streaming; late data + `withWatermark`.
9. Exactly-once Kafka → Spark/Flink → Delta/Iceberg.
10. Snowflake's multi-cluster, shared-data architecture (three layers; virtual warehouses; FoundationDB-backed metadata; micro-partitions ~16 MB compressed).
11. Why is Snowflake's pruning faster than Hive partition pruning? (Per-micro-partition min/max + clustering depth; no S3 listing.)
12. Migrate a Hive table with 200 K partitions to Iceberg.
13. Schema evolution: rename/add/drop/reorder in Iceberg/Delta — what breaks?
14. Bronze/Silver/Gold medallion pipeline — when to materialize, when to view.
15. Photon internals: vectorized C++ engine, SIMD, columnar batches; why faster than JVM Spark.

**Grill-down example.** "Spark shuffle bottleneck." → "Switch to broadcast if one side fits; AQE auto-detects; salt skewed keys; partition-by sort-merge join on already-partitioned columns; reduce shuffle partitions; enable shuffle service."

**Sample expert outline — "Petabyte clickstream lakehouse":**
- Ingest: Kafka → Spark Structured Streaming → Iceberg bronze (raw events, hourly hidden partition transforms).
- Silver: dedup on `(user_id, event_id)` with watermark + state TTL; dimension joins; COW for low-frequency updates, MOR + deletion vectors for high-frequency.
- Gold: pre-aggregated hourly/daily facts; clustered on common predicates; Iceberg REST catalog exposed to Trino/Snowflake/BigQuery.
- Maintenance: scheduled `OPTIMIZE` + `VACUUM` (Delta) or `REWRITE_DATA_FILES` + snapshot expiration (Iceberg).

**Common traps.** Confusing Delta's `OPTIMIZE` (file compaction) with `VACUUM` (snapshot retention). Not knowing Iceberg's hidden partitioning means users query the *raw* timestamp, not a partition column. Forgetting that S3 listing is the metadata-scale bottleneck that both Iceberg and Delta fix.

**What separates strong.** Articulating the *physical* layout (manifest → manifest-list → snapshot → metadata.json for Iceberg) and the *concurrency control* (Iceberg OCC on the metadata pointer atomic-swap; Delta similar).

---

### C) AI / ML Infrastructure (Anthropic, OpenAI, Google DeepMind, Mistral, xAI, Hugging Face, Scale AI, Databricks Mosaic)

**What they emphasize.** Vector DBs (HNSW vs IVF, recall/latency/memory trade-offs), RAG architectures (chunking, hybrid retrieval, re-ranking), embedding storage at billion+ scale, feature stores (Feast vs Tecton vs Databricks FS), ML metadata (MLflow), data versioning (DVC, LakeFS, Iceberg snapshots), training data pipelines, model serving DB interactions.

**High-signal questions actually asked** (Hello Interview, IGotAnOffer's OpenAI guide, Interview Query's Anthropic guide, ByteByteGo's Generative-AI System Design book, DataInterview, datatalks.club):

1. Design a RAG system for an enterprise knowledge base (chunking; embedding model; vector DB; hybrid retrieval; re-ranker; prompt assembly; cost/latency).
2. Hello Interview's verbatim RAG prompt: *"Design an AI-powered tutoring platform that uses Retrieval-Augmented Generation (RAG) to help students get answers to questions about their assignments and coursework. The system should be able to retrieve relevant educational content and generate contextual responses to student queries."*
3. HNSW vs IVF — when each, at 1 M / 100 M / 1 B vectors?
4. Why is HNSW memory-heavy and what knobs (M, efConstruction, efSearch) reduce it without killing recall? What does PQ buy you?
5. Design an inference gateway that routes between GPT-4 / GPT-4 Turbo / GPT-3.5 with different latency and cost profiles (DataInterview's reported prompt).
6. Feature store for online + offline parity. (Online: Redis/DynamoDB; offline: Delta/Iceberg/BigQuery; point-in-time correctness; training-serving skew.)
7. Vector search with metadata filtering: pre-filter vs post-filter — when does each win?
8. Scale vector DB from 10 M to 10 B vectors. (Sharding by hash of vector ID; per-shard HNSW; query fan-out + top-k merge; quantization tiers; disaggregated storage à la Milvus 2.x.)
9. Data versioning for training: reproduce a 6-month-old model. (Iceberg snapshot ID + DVC/LakeFS pointer + container SHA + RNG seed; lineage in MLflow.)
10. Design the metadata store for an ML platform (runs, params, metrics, artifacts, lineage; Postgres + S3 + Iceberg).
11. Billion-scale embedding index. Databricks' own engineering post is candid: *"Most vector indexing libraries — FAISS, ScaNN, Annoy — assume all your data fits on a single machine. That works at tens of millions of vectors. At a billion vectors with 768-dimensional embeddings, you are looking at terabytes of raw floating-point data before you even start building an index."*
12. Hybrid search (BM25 + dense + RRF fusion). Where do sparse and dense indices live? How do you re-rank with a cross-encoder?
13. RAG correctness/eval: recall, faithfulness, answer correctness. (Ragas, TruLens, golden eval sets.)
14. Real-time RAG freshness: a doc updated 30 s ago must be retrievable. (CDC → embedding worker → vector-store upsert; or scheduled re-index; trade-off vs cost.)
15. Cost estimation: OpenAI embeddings + Pinecone for 50 M docs, 100 RPS, p99 < 100 ms.

**Grill-down.** Interview Query's Anthropic ML Engineer guide states the bar: *"You are evaluated on engineering judgment under real production constraints, explicit tradeoff reasoning around latency, GPU utilization, cost, evaluation pass rates, and system reliability."*

**Sample expert outline — "Production RAG":**
- Ingestion: docs → cleaner → semantic chunker (300–800 tokens, 10–15 % overlap) → embedder (text-embedding-3-large or BGE-large; batched serving cluster; idempotent on doc hash) → vector DB upsert + BM25 update + metadata KV.
- Storage: HNSW (M=32, efC=200) in Milvus/Weaviate/pgvector for ≤ 100 M; sharded HNSW or IVF-PQ + DiskANN at billion scale; S3 cold for raw doc + chunks.
- Query: rewrite (HyDE / multi-query) → parallel dense + BM25 → RRF fusion top-50 → cross-encoder re-rank → top-K (5–10) → prompt assembly with citations.
- Eval: golden set + LLM-as-judge faithfulness + retrieval recall@K weekly; CI breaks on regression.
- Freshness: Debezium-style CDC from source-of-truth Postgres → embedding worker → upsert; soft-delete + tombstone for removals.
- Cost: cache (query, top-K) for repeats; tiered embeddings (small for index, large for re-rank); batch nightly re-embed only on model upgrade.

**Common traps.** Building a "vector DB" without metadata filtering and tenancy isolation; not measuring recall; assuming dense > sparse always (citations and rare entities favor BM25); ignoring that the embedding model dominates retrieval quality more than the index does.

**What separates strong.** Naming the eval discipline; quantifying p99 across the entire pipeline; understanding pre-filter vs post-filter mechanics; defending pgvector for ≤ ~5 M vectors against a reflexive "use Pinecone."

---

### D) Finance / Quant / Consulting (Goldman Sachs, JPMorgan Chase, Jane Street, Citadel, Bloomberg, Two Sigma, Bank of America)

**What they emphasize.** Time-series at scale (kdb+/q, TimescaleDB, custom tick stores), transactional consistency (ledger correctness, audit trails), low-latency systems, decimal-precision arithmetic, regulatory retention (SEC 17a-4 WORM, MiFID II), data lineage, recovery semantics under partial failure.

**High-signal questions actually asked** (TimeStored kdb+ interview guide, Glassdoor Citadel/Bloomberg/Jane Street/Goldman reports, DataLemur, Exponent JPMorgan guide, interviewing.io Bloomberg guide, Prepfully):

1. (kdb+/q, asked at any tick-data shop, verbatim from TimeStored) *"In a kdb tick setup. How does the RDB recover if the machine suffers a restart? What are the steps to appending data to a partitioned table? What are some of the functions you would use? What is kdb query functional form? When would you use it? … Describe a failure scenario, e.g. out of RAM and the steps to fully recover? … I don't have enough RAM for the data I want to store. What can I do? End users are subscribing to the TP and the TP can't keep up?"*
2. (Verbatim TimeStored futures/options scenario) *"I have a data set with 1000 instruments, receiving 1000 rows per second, how much storage and RAM will I need to capture this? If I mostly want to perform a query to find 30 days of history for a single instrument, how should I structure the system? … later I have another important query that needs to run fast, the highest price each day for the last week and today for 10 instruments? How can I make that query fast?"*
3. (TimeStored verbatim) *"Name all kdb attributes? When would I use each? Name all the kdb adverbs?"* (`` `s# `p# `u# `g# `` — sorted, parted, unique, grouped — and what each buys you on `?` lookups and joins.)
4. (Glassdoor verbatim) *"1. query max value while still having all columns 2. aj lj ij join functions 3. sym file d file and how to maintain db way more big than memory."* (`aj` = as-of join = prevailing quote at trade; `lj` = left join; `ij` = inner.)
5. Bloomberg system design (Glassdoor verbatim 2026): *"The question involved designing a system to store and search incoming stories with timestamps, which on the surface sounds practical."*
6. Bloomberg / interviewing.io verbatim: *"System Design questions at Bloomberg tend to skew practical and tend to be finance-related. For instance, you might be asked questions such as: How would you design a stock exchange?"*
7. JPMorgan (Exponent's reported system-design list, verbatim): *"Design a real-time fraud detection system for a banking application. Design a rate-limiting system for financial transaction APIs. Design a time-series database for storing financial market data. Design a real-time notification system for banking alerts and transactions. Design a role-based access control system for financial institutions."*
8. Goldman Sachs SQL (DataLemur verbatim): *"Goldman Sachs is interested in analyzing data regarding client transactions in order to uncover insights for their investment strategies. Specifically, they want to track the total investment amount in each asset (stocks, bonds, derivatives), from each client per month. … Design a database schema suitable for this problem and write a PostgreSQL query to provide a summary of the total invested amount in each asset type, by each client, per month."*
9. Bank of America SQL (DataLemur verbatim, see §3 above).
10. "Decimal vs floating point for money — show me what goes wrong with `FLOAT`." (`0.1+0.2 ≠ 0.3` in IEEE-754; accumulated drift; switch to `NUMERIC(p,s)` or integer cents/basis points.)
11. "Design an immutable, append-only ledger with audit trail (SOX/MiFID-compliant)." (WORM storage, hash-chained log lines, separate read-model via CQRS, no in-place updates ever.)
12. "Order-matching engine schema + concurrency." (In-memory price-time priority book; persistent journal of every event before ACK; deterministic replay.)
13. "T+1/T+2 settlement reconciliation." (Saga across custodian/broker/exchange feeds; idempotent compensation.)
14. "Why is kdb+ faster than PostgreSQL for tick data?" (Column-store on disk via splayed/partitioned tables; mmap; vectorized q array primitives; `aj` O(log n) over sorted partitions.)
15. Citadel Market Data Developer (Glassdoor verbatim): *"computer science; finance; C++; programming; coding; brain teaser; need to know what's going on in the market; need to know C++ from top to bottom; need to know UNIX pretty well; need to know SQL pretty well. need to write efficient code quickly; need to recognize what the code snippet is doing."*

**Grill-down.**
- "Your ledger gets a duplicate event." Idempotency key on every write; `UNIQUE` on `(source, source_event_id)`; retry-safe.
- "Regulator asks for the state of account X at 14:30 on March 8, 2023." Event-sourced ledger → replay to timestamp; or Iceberg/Snowflake time travel; or temporal tables with `SYSTEM_VERSIONING`.
- "Your tick database lost 3 s of data after a switch flap." Recover from upstream exchange's resend; verify with sequence-number gap detection; reconcile post-hoc.

**Sample expert outline — "Time-series store for market data":**
- Ingest: exchange multicast → kernel-bypass NIC (Solarflare/Mellanox) → C++ feedhandler → in-memory ring buffer → tickerplant (TP) writing to RDB (real-time DB, in-memory) and a sequential journal on NVMe.
- End-of-day: RDB rolls into HDB (historical DB), splayed/partitioned by `date`, sorted by `sym` with `` `p# `` attribute on `sym`.
- Storage: ~10 B ticks/day × ~32 B/tick after compression ≈ 320 GB/day; 7 years retention ≈ 800 TB on tiered storage (hot NVMe last 30 d, warm SSD last year, cold object store rest).
- Query: q's `aj` for prevailing quote at trade; partition pruning by date; per-sym attribute makes `sym` equality ~O(1).
- HA: TP writes journal to two NVMe drives; secondary TP replays journal; cross-DC async replication for DR.
- Audit: every tick immutable; WORM-archive nightly to S3 with object-lock + SHA-256 manifest.

**Sample expert outline — "Immutable ledger":**
- Tables: `journal(event_id PK, account_id, amount_minor, currency, kind, ts, idempotency_key UNIQUE, hash, prev_hash)` — hash-chained; `accounts_current(account_id PK, balance_minor, version)` — derived.
- Writes: parameterized SQL in `SERIALIZABLE` (Postgres SSI) txn — INSERT journal row, UPDATE balance row `WHERE version = ?`, fail and retry on conflict.
- Money: `NUMERIC(18,4)` or `BIGINT` in minor units; never `FLOAT`.
- Read model: replicas for reports; materialized views per regulatory line item.
- Audit: pgaudit + WAL archive + offsite WORM; quarterly hash-chain verification job.

**Common traps.**
- `FLOAT`/`DOUBLE` for money.
- Allowing updates/deletes on the journal table.
- Ignoring sequence-number gaps in market data feeds.
- Audit log mutations are not themselves auditable.
- Quoting p99 but not p99.9 (the tail is what regulators see).

**What separates strong.** Quantifying in microseconds; naming kdb+ `aj` for tick joins; defaulting to `SERIALIZABLE` for ledger writes and *explaining why* (write-skew between balance check and update); knowing that Bloomberg/Citadel probe C++/UNIX/SQL all three, not just one.

---

### COMMON ACROSS ALL VERTICALS

20 questions you must be ready to answer in any DB-adjacent interview:
1. Explain ACID. Then explain how each letter changes under MVCC.
2. Compare B+Tree and LSM-Tree. When is each preferable?
3. Clustered vs non-clustered (secondary) index? How does it differ between InnoDB (clustered PK) and Postgres (heap + index)?
4. Walk through `EXPLAIN ANALYZE` on a query I'll give you.
5. What is a deadlock? How does a DB detect / prevent it?
6. Four ANSI isolation levels and the anomalies each allows. Where does snapshot sit, and what does it still allow?
7. Why is `SELECT COUNT(*)` slow in Postgres but fast in MyISAM? (MVCC vs not.)
8. CAP vs PACELC — where does each database fall?
9. Walk through Raft.
10. Why is consistent hashing useful, and how does virtual-node count affect skew?
11. How does Postgres replication work? Sync vs async trade-offs?
12. Connection pooling — why and how? (PgBouncer transaction-mode quirks; max-conn vs max-cores.)
13. N+1 query problem and three ways to fix it.
14. Design a schema for a chat app / e-commerce / booking system / blog (pick one and own the trade-offs).
15. Back up a 10 TB Postgres for PITR.
16. Slow-query debugging session start to finish.
17. SQL injection: one example, three preventions.
18. Partial / covering / composite indexes.
19. Why is `LIMIT … OFFSET` slow at deep offsets, and what's the keyset-pagination fix?
20. JSON in a relational DB — when is it justified and what are the costs?

---

## CROSS-LINKS TO OTHER CS AREAS

- **Operating systems**: buffer pool ↔ page cache; WAL ↔ journaling filesystems; concurrency ↔ locks/futexes; isolation ↔ memory consistency models; sequential I/O ↔ disk scheduler; mmap vs explicit I/O.
- **Networking**: replication latency ↔ RTT; consensus rounds ↔ message complexity; protocol design (PostgreSQL wire protocol); TLS in-transit; HOL blocking in TCP vs QUIC for streaming.
- **Data structures**: B-tree/B+Tree, LSM, skip list (Memtable), Bloom filter (LSM read path), HyperLogLog (ndistinct), Count-Min Sketch (top-K), HNSW graph, vector clocks, Merkle tree (anti-entropy).
- **Compilers**: parser → AST → optimizer → codegen identical to a DB query pipeline; Cascades framework; vectorized execution and SIMD codegen in Photon/DuckDB; Spark Whole-Stage Codegen.
- **Distributed systems**: FLP, CAP, Lamport clocks, vector clocks, HLC (Cockroach/YugabyteDB), Paxos/Raft/EPaxos, gossip protocols, anti-entropy.

---

## CANONICAL READING ORDER

1. CMU 15-445 lectures (Pavlo, Fall 2023 or later) — watch end to end.
2. Petrov, *Database Internals*, Part I.
3. Kleppmann, *Designing Data-Intensive Applications*, cover to cover.
4. Petrov, *Database Internals*, Part II.
5. *Readings in Database Systems* 5e (Red Book) — editors' commentary first, then chapter papers.
6. Core papers in order: Codd 1970 → System R (Selinger 1979) → ARIES (Mohan 1992) → GFS (2003) → MapReduce (2004) → Bigtable (2006) → Dynamo (2007) → Spanner (2012) → Snowflake (SIGMOD 2016) → Raft (USENIX ATC 2014) → Lakehouse (CIDR 2021) → Photon (2022).
7. CMU 15-721 (graduate, in-memory + columnar + advanced topics).

---

## WHAT'S OVERRATED vs UNDERRATED IN INTERVIEWS

**Overrated (do not over-invest):**
- Memorizing every NoSQL product's marketing differentiation.
- Reciting CAP without naming a real system.
- Deep ARIES log-record formats (know the three phases + steal/no-force; the rest is bonus).
- Hyper-niche tuning flags (`work_mem`, `effective_cache_size`) — say you'd profile first.

**Underrated (massive interview leverage):**
- Window functions and CTEs at fluency level.
- Postgres `EXPLAIN ANALYZE` + extended statistics story.
- Snapshot vs Serializable + write skew (name the doctor-on-call example).
- Saga and idempotency keys.
- Citing engineering blogs (Discord, Uber, Figma, Notion, Stripe) — interviewers love this.
- Articulating *why* you'd pick X.

---

## WHAT CHANGED IN THE LAST 2–3 YEARS (2023–2026)

- **Vector DBs went mainstream.** HNSW + IVF are now standard interview content for any AI-adjacent role. pgvector merged HNSW (2023); Pinecone went serverless; Milvus 2.x disaggregated compute/storage.
- **Lakehouse format war effectively settled.** Iceberg is now the de facto open standard (AWS S3 Tables 2024; Snowflake + BigQuery support it; Databricks UniForm bridges Delta↔Iceberg). Databricks reportedly paid nearly $2 B for Tabular in June 2024 (TechCrunch citing Bloomberg).
- **Serverless databases**: Aurora Serverless v2, Neon (serverless Postgres with branching), PlanetScale (Vitess, branching), Snowflake unistore — separation of compute/storage is now the default mental model.
- **HTAP matured**: TiDB+TiFlash, SingleStore, Aurora dual-storage.
- **AI-driven query optimization**: OtterTune (Pavlo) and learned cardinality estimators are appearing in products (Microsoft, Oracle).
- **Databricks scale**: per Databricks' own Sept. 8, 2025 press release, the company *"crossed a $4 billion revenue run-rate during Q2"* of FY2026 (up from the $2.4 B run-rate reported by CNBC in mid-2024) — illustrating the speed of the lakehouse market.
- **Discord's 2022 Cassandra→ScyllaDB migration** ("How Discord Stores Trillions of Messages," March 2023) became a canonical case study for hot partitions, JVM GC, and leaderless wide-column tuning.
- **RAG architecture** became a standard system-design prompt at AI labs.

---

## HOW TO SET THIS UP AS A CLAUDE PROJECT

**Project knowledge to upload (one-time).**
1. This primer (as a Markdown file).
2. Your personal weak-area list — be specific ("write skew," "Iceberg manifest list," "Raft safety property").
3. A few canonical PDFs you'll reference often: the Kleppmann DDIA chapter you're in, ARIES, Dynamo, Snowflake SIGMOD, Raft.
4. A schema dump from any real database you work on (anonymized) so Claude can give grounded feedback.
5. A "target company" list with 3–5 prioritized companies and their known interview formats.

**Project system instructions (paste into project setup).**
> You are my DBMS interview coach. Behaviors: (a) Default Socratic — ask one probing question before answering. (b) On a system-design prompt, push me to clarify scale, read:write ratio, latency budget, consistency need *before* drawing boxes. (c) When I propose an answer, grill me with at least two follow-ups: "what breaks at 10× scale?" and "what's the alternative and why didn't you pick it?" (d) When citing a system, name the canonical paper. (e) Refuse to give me the answer until I've taken a shot. (f) For SQL, return both `EXPLAIN` annotation and a rewrite. (g) Mark every concept "core / boundary / niche" so I know what to spaced-repetition.

**Daily workflow with Claude.**
- **Spaced repetition.** "Generate 10 cloze cards on ARIES recovery, mixed difficulty, format `Q :: A`." Review tomorrow + day 3 + day 7.
- **Mock interviews.** "Be a Meta staff engineer. Run a 45-minute system-design interview on 'design Instagram Reels feed.' Push back at staff level. Score me at the end on clarifying questions, scale awareness, schema, indexes/sharding, caching, replication, failure modes, communication."
- **Query-plan review.** Paste `EXPLAIN (ANALYZE, BUFFERS) …` output. "Diagnose. Three specific rewrites with expected impact."
- **Schema-design feedback.** Paste your DDL. "Review for 3NF/BCNF violations, missing indexes (esp. FK indexes in Postgres), hot-row risk, audit-trail gaps. Suggest two evolutions if write volume grows 100×."
- **Paper summaries.** "Summarize the Dynamo paper in 400 words; then quiz me with 5 questions a Google L5 interviewer would ask."
- **Vertical drills.** "I have a Citadel C++ market-data interview tomorrow. Run a 30-minute focused drill on UNIX, SQL, kdb+ basics, brain-teasers, and low-latency C++."
- **Trade-off arguments.** "Argue both sides: 'pgvector vs Pinecone for a 20 M-doc internal RAG.' Tell me which side a senior IC at Anthropic would pick and why."

**Anti-patterns to avoid.**
- Don't let Claude write the answer first; you'll feel competent and fail in person.
- Don't accept hand-waves; demand the canonical paper / blog post / verbatim grill-down.
- Don't confuse "I've read the book" with "I can whiteboard Raft in 10 minutes" — drill the second weekly.

---

## RECOMMENDATIONS (Staged)

**4 weeks until an interview:** finish Phase 1 + scan Kleppmann ch. 5–9; drill 30 system-design prompts; 50 LeetCode SQL Medium/Hard; 6 mocks.

**3 months:** complete Phases 1–3 + the BusTub projects + Raft lab; all of Kleppmann; 8 canonical papers; weekly mocks in the last month.

**6+ months:** all the above + Phase 4 + Phase 5 vertical project + 2–3 OSS contributions to a real DB / data platform (TiKV, DuckDB, Cockroach, Trino, Iceberg). This produces a candidate that interviewers compete to hire.

**Benchmarks that should change your plan:**
- Can't explain MVCC in 3 minutes after Phase 1 → re-do storage internals.
- Raft lab passes all tests → you're ready for distributed-systems system-design rounds.
- A senior friend gives you ≥ 4/5 on a mock → schedule real interviews.
- Same grill-down twice ("what breaks at scale?") without a crisp answer → you're memorizing, not reasoning. Stop and do trade-off drills.

---

## CAVEATS

- Glassdoor / Blind interview questions are self-reported and anonymized; treat them as *style* signals, not verbatim guarantees. Question banks evolve.
- Vendor blog posts (Pinecone, Databricks, Snowflake) are commercial and bias toward their products; their technical primitives (HNSW mechanics, Snowflake micro-partition layout) are reliable, but their comparisons are not neutral.
- The Iceberg vs Delta vs Hudi landscape is moving fast — by late 2026, format-bridging (UniForm, XTable) may make the choice less consequential than it is today.
- AI-infra interview reports are sparser than FAANG/finance reports; the AI-lab vertical content here leans more on platform docs (Pinecone, Databricks, Hello Interview) and role-specific prep guides (Interview Query Anthropic, IGotAnOffer OpenAI) than on verbatim candidate trip reports.
- LLM-generated interview question lists sometimes hallucinate prompts; cross-check against ≥ 2 named sources before treating a prompt as canonical.
- Discord's Cassandra→ScyllaDB metrics (177→72 nodes; p99 reads 125 ms→15 ms; trillions of messages) come from Discord's own engineering blog and Bo Ingram's talks; production numbers do not generalize to your workload.
- Databricks/Snowflake revenue claims are time-sensitive — Databricks's run-rate moved from ~$2.4 B (mid-2024, CNBC) to $4 B+ (Sept. 2025 Databricks press release); check the latest before citing in an interview.