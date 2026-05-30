# The Senior Engineer's Study Guide: OOP, LLD, HLD, and SDK Building for Top-Company Interviews

## TL;DR
- **For pure depth on distributed/HLD, Arpit Bhayani's System Design Masterclass (₹49,999 / 6 weeks, 12 sessions, 28 systems, 16+ patterns) is the strongest single resource for SDE-2+; for LLD/OOP, supplement it with `ashishps1/awesome-low-level-design` and Refactoring.Guru — Arpit's own masterclass FAQ explicitly states "it will not cover writing and designing classes, and low-level design patterns,"** so a two-track plan is required.
- **The senior-vs-junior delta in interviews is the same across FAANG, AI labs, and SaaS giants**: at L4/E4/SDE-1 a clean high-level box diagram earns "hire"; at L5+/SDE-2+ you must proactively own failure modes, name technologies with justification, do back-of-the-envelope numbers, and drive the conversation — at Staff (L6), waiting for the interviewer to ask follow-ups is itself the failure signal.
- **The interview shape is bifurcating in 2026**: classical FAANG (Google/Meta/Amazon) tests scalable web systems; AI labs (Anthropic, OpenAI) test GPU-batched inference and async-to-sync mapping; payments/SaaS (Stripe) tests API design with idempotency and ledgers; quant/finance (Citadel, Jane Street) tests C++/OCaml depth and order-book systems; and consulting tech (McKinsey QuantumBlack) blends pair-programming with classic business cases. Optimize prep for your target.

---

## Key Findings

### 1. Arpit Bhayani's Courses — What You Actually Get

Arpit Bhayani — per his About page on arpitbhayani.me (May 2026), *"Currently, I am a Principal Engineer II at Razorpay, working at the intersection of Data and AI, and building Agent Studio… Previously, I was a Staff Engineer at Google, where I worked on GCP Memorystore and GCP Dataproc."* He is also building **DiceDB**, described on his site as *"a fork of Valkey with multi-tiering and query subscriptions"* (note: an older YouTube About page still calls it "a re-implementation of Redis in Go" — the arpitbhayani.me phrasing is the authoritative current one). He runs **three** courses through Relog Deeptech Pvt. Ltd.:

**(a) The System Design Masterclass — ₹49,999 ($649) live cohort / ₹39,999 ($599) recordings-only.** This is the ~50k INR course referenced in the question. Format: 6 weeks, 12 sessions, ~50+ hours, Saturday/Sunday 9 AM–12 PM IST, taught entirely by Arpit (no TAs). Target audience: SDE-2, SDE-3, and above with ≥2 years experience. Stated coverage: **28 systems, 16+ patterns**. The actual week-by-week syllabus (verbatim from arpitbhayani.me/masterclass/, June 2026 cohort):

- **Week 1 — Foundations:** Online/Offline indicator design; 3 mental models for system design intuition; Connection pools and DB Proxies; Caching issues at scale; Async processing and Kafka essentials; Communication paradigms and Deployment Log Streamer.
- **Week 2 — Databases and Locking:** Pessimistic vs Optimistic Locking; Remote Locks and Distributed Locks; Columnar, Graph, and Wide Column stores; Designing Slack's real-time text communication; Scaling WebSockets.
- **Week 3 — Going Distributed and Social:** Horizontally-scalable Load Balancers (avoiding SPoF); Leader Election; Consistent Reads; Role of CDN in Live Streaming; Photos Upload at scale; Private Photos and Gravatar; HashTag Counter Service.
- **Week 4 — Storage Engines:** Redis Internals; operational complexity of consistent hashing; No-downtime data migration; Designing a Word Dictionary without a DB; Designing S3.
- **Week 5 — High Throughput Systems:** Multi-tiered Orders for Amazon; LSM Trees ground-up; Event Ingestion at Scale; Distributed ID Generators; Three ways to handle Hot Shards.
- **Week 6 — High Volume Systems:** Cricbuzz's Text Commentary; Distributed Task Scheduler; Flash Sale design; Impressions Counting at scale; GeoSpatial Search.

**Signature topics** that show up repeatedly in his content and his "Asli Engineering" YouTube channel (subscriber count varies between sources — flagged as unverified): consistent hashing operational pitfalls, Bloom filters, LSM trees, Bitcask/append-only logs, Redis internals (he wrote DiceDB), distributed task schedulers, hot-shard handling, leader election, no-downtime migrations, and storage engine design from first principles. Reviewers (e.g., Bharath Chandra Elluru) consistently call out *"From database internals to WebSockets and Bloom filters, I've moved beyond surface-level theory."*

**Critical caveat for this question:** Arpit's own FAQ on the masterclass page states unambiguously — *"The course will cover some aspects of Database Design and its internals, but it will not cover writing and designing classes, and low-level design patterns. The course is typically aimed at covering the massive spectrum of System Design and Software Architecture."* If you need LLD, look elsewhere.

**(b) System Design for Beginners** — self-paced with bi-weekly doubt-solving sessions; assumes no prior knowledge; mutually exclusive with the Masterclass (minimal overlap).

**(c) Redis Internals** — self-paced, hands-on, re-implements Redis data structures, algorithms, and core features in Go. This is the closest Arpit comes to LLD/OOP-style work.

He does **not** offer a dedicated OOP or LLD course. The GitHub repo `arpitbbhayani/system-design-questions` contains the assignment problem statements used in the masterclass.

### 2. OOP — Industry Depth

**SOLID with real production violations.** Useful patterns to memorize for interview answers:

- **SRP violations** look like "UserAccountManager" classes that handle authentication *and* activity logging — the test is whether you can name two distinct *reasons to change*. Real example surfaced in open-source: NLog's `Target` base class historically pushed layout concerns into the base abstraction, forcing every `Target` subtype to know about layouts whether they need them or not.
- **OCP violations** show up as enum/if-elif cascades for "renderer types" or "notification types" that require modifying the original class for every new variant. Fix: register implementations through a factory dictionary.
- **LSP violations** are the most insidious because they compile fine but fail at runtime. Canonical industry example: a `Bird` interface with `fly()` that `Penguin` cannot honor → forces `NotImplementedException`. The deeper LSP rule applies at **API boundaries**: if v2 of a REST API changes `created_at` from Unix timestamp to ISO 8601 while keeping the same field name, that is an LSP violation against v1 clients. Senior interviewers (Stripe is the canonical example) probe this directly.
- **ISP violations:** the "fat interface" with `work()` and `eat()` forcing `Robot` to implement `eat()`. Split into `Workable` and `Eatable`.
- **DIP violations:** `OrderService` instantiating `StripeGateway()` directly; fix by depending on a `PaymentGateway` interface and injecting the concrete via constructor. Dependency injection is the *technique*; DIP is the *principle*.

**Anti-pattern: dogmatic SOLID.** Production teams report that 50-class micro-fragmentation is worse than three coherent god-classes. The pragmatic rule: extract responsibilities when maintenance pain justifies the indirection, not on day one.

**Design patterns — the production-frequency tier list** (from real Python/Java codebases):

- **Tier 1 (you will use these weekly):** Factory Method, Strategy, Observer, Adapter, Decorator, Singleton (cautiously — module-level state is often cleaner in Python).
- **Tier 2 (regular but situational):** Builder (for objects with 5+ optional constructor args, fluent SDK builders), Template Method, Chain of Responsibility (middleware), State (for explicit FSMs), Proxy (lazy loading, ORMs).
- **Tier 3 (read about, rarely write):** Abstract Factory, Bridge, Composite, Flyweight (still vital for memory-intensive systems — game engines, large text editors), Mediator, Memento, Visitor.

Real-world Python sightings: Django form factories, SQLAlchemy dialect factories, the `unittest` `TestLoader`, RxPy observer chains, and the OpenAI Python SDK's resource hierarchy (each `client.chat.completions.create(...)` is a method on a lazily-instantiated resource property — a textbook Facade over the BaseClient).

**OOP vs Functional vs Procedural — tradeoffs:** OOP excels when you have *state with identity* and *polymorphic behavior* (UI components, game entities, ORM models). Functional excels for *data pipelines* (map/reduce, ETL), *concurrency* (immutability eliminates a class of bugs), and *math-heavy code*. Procedural still wins for *scripts* and *bootstrap code* where 5 functions are clearer than 5 classes. Senior signal: knowing when to *not* use OOP. Python's `dataclass` + module-level functions often beats a hierarchy of classes.

**How OOP is tested at top companies:**
- **Google**: rarely a pure "OOP/LLD" round; OOP appears inside coding rounds as "design the classes for this small problem" sub-question.
- **Meta**: similar to Google, plus an optional product-architecture round for senior candidates that probes class boundaries.
- **Amazon**: LLD is a first-class round at SDE-2+ (commonly called "OOD") — parking lot, library, ATM are real questions.
- **Stripe**: their famous API design round *is* OOP at the boundary — designing `Charge`, `Refund`, `Subscription` as composable resources with idempotency keys.
- **Databricks**: Low-Level System Design with pseudocode in Google Docs, more hands-on than typical FAANG OOD.

### 3. LLD — The 45-to-60-Minute Format

The format is consistent across companies that ask it:

1. **Clarify (3–5 min):** vehicle types, multi-floor, payment model, concurrency. *Most candidates skip this and over-engineer.*
2. **Identify entities (5–8 min):** nouns → classes (`ParkingLot`, `ParkingFloor`, `ParkingSpot`, `Vehicle`, `Ticket`); verbs → methods. Use enums (`VehicleSize`, `SpotType`).
3. **Apply SOLID + patterns (10 min):** Strategy for spot-allocation (nearest-to-entrance vs first-available vs floor-based); Factory for spot/vehicle creation; Singleton for the `ParkingLot` if justified (drop it if the interviewer pushes back); Observer if displays need real-time updates.
4. **Code core flows (20 min):** focus on the entry/exit happy path; pseudo-code is fine for less-important methods.
5. **Concurrency follow-up (5–10 min):** "now make it thread-safe" — lock at the spot level, not the lot level; mention deadlock avoidance.

**The standard problem set, ranked by frequency** (drawn from Hello Interview, awesome-low-level-design, Coudo AI, CodeZym question banks):

- **Tier 1 (must-know cold):** Parking Lot, LRU/LFU Cache, Rate Limiter (token bucket + leaky bucket), URL Shortener, Snake & Ladder, Vending Machine.
- **Tier 2 (high frequency):** Splitwise, BookMyShow (movie booking), Elevator System, ATM, Library Management, Chess.
- **Tier 3 (senior-level / domain-specific):** Cab Booking (Uber dispatch with strategy pattern for matching), Social Media Feed (timeline generation + fan-out tradeoff), Hotel Booking (concurrency + idempotency).

**The senior-vs-junior signal in LLD:** Juniors write the classes correctly. Seniors articulate *why* — "I'm choosing Strategy here because the business is likely to add new pricing models per vehicle type, and we don't want to modify the core `PricingService` for each variant." That single sentence flips the score.

### 4. HLD — Format, Framework, and Senior Signals

**The universal HLD flow (45–60 min):**

1. **Requirements (3 min):** functional (the *what*) and non-functional (the *how well* — latency, availability, consistency).
2. **Capacity estimation / back-of-envelope (5 min):** DAU → QPS → storage/yr → bandwidth.
3. **High-level architecture (10 min):** boxes for clients, LB, API gateway, services, DBs, cache, queue, CDN.
4. **Deep dives (25 min):** **this is where L4 separates from L5+** — interviewer expects you to proactively pick 2–3 components to drill into.
5. **Trade-offs and failure modes (7 min):** what breaks, how you detect, how you recover.

**Capacity estimation worked example — Design Twitter (the Jeff Dean / ByteByteGo template):**

- Assumed numbers (state them aloud — "These are not real Twitter numbers"): 300M MAU; 50% DAU → 150M DAU; 2 tweets/user/day → 300M tweets/day.
- Tweets/sec: 300M / ~100,000 sec/day ≈ **3,000 TPS average, ~6,000 TPS peak** (2× rule for events).
- Media: 10% tweets contain media @ 1MB avg → 30M × 1MB = 30 TB/day → ~55 PB over 5 years (this matches reasoned industry estimates).
- Cache: 500 tweets × 200 bytes/tweet = 100 KB per user × 165M ≈ **16 TB Redis** total; with 3× replication ≈ 48 TB.

The arithmetic mistake to avoid: not collapsing seconds-in-a-day to ~100,000. Precise division is a trap; interviewers want order-of-magnitude.

**The Top-12 HLD question bank you must internalize:**

| Problem | Crux to nail | Common follow-up |
|---|---|---|
| Design Twitter/X | Fan-out on write vs fan-out on read; celebrity-user hybrid | Search; trending |
| Design YouTube | CDN, transcoding pipeline, chunked HLS/DASH | Live streaming |
| Design WhatsApp | WebSocket scaling; E2E encryption key handshake; message ordering | Group messages |
| Design Uber | Geospatial indexing (S2/QuadTree/Geohash); driver-rider matching | Surge pricing |
| Design Netflix | CDN edge selection; pre-positioning; ABR | Recommendations |
| Design Google Search | Inverted index; PageRank; query routing; index sharding | Auto-complete |
| Design Notification System | Multi-channel routing; rate limiting per user; deduplication | Priority queues |
| Design Payment System | Idempotency keys; ledger double-entry; saga for failures | Multi-currency |
| Design Rate Limiter | Token bucket vs sliding window; Redis Lua atomicity | Distributed counters |
| Design Distributed Cache | Consistent hashing ring; replication; cache stampede | Hot key |
| Design Message Queue | At-least-once vs exactly-once; partitioning; consumer groups | Backpressure |
| Design Distributed File System | Chunking; replication factor; metadata service | Reed-Solomon |

**Key components every senior must reason about (not just name):**

- **Load Balancer:** L4 (TCP, faster) vs L7 (HTTP-aware, can do canary by header). Algorithms: round-robin, least-connections, consistent hashing for sticky sessions. **HAProxy / Nginx / Envoy / AWS ALB** — interviewers want you to pick one and justify.
- **CDN:** push vs pull; TTL/cache invalidation; edge-vs-origin shielding. **CloudFront, Akamai, Fastly, Cloudflare**.
- **API Gateway:** auth, rate limiting, request transformation. Kong, AWS API Gateway, Envoy.
- **Message Queues:** **Kafka** (log, partitioned, ordered within partition, durable, high throughput) vs **RabbitMQ** (smart broker, classical pub-sub, complex routing, lower throughput). Pick the right one and say why.
- **SQL vs NoSQL:** "It depends on the access pattern" is junior. Senior: "Cassandra for write-heavy append-only logs with tunable consistency per query; PostgreSQL for OLTP with multi-row transactions; DynamoDB when I want a managed service and can model around single-table; ScyllaDB if I'm Cassandra-shaped but latency-bound."
- **Cache:** Redis (data structures, persistence, Lua scripting) vs Memcached (simple, slab allocator, multi-threaded). For interview, Redis is almost always the right answer unless the interviewer constrains you.
- **Search:** Elasticsearch / OpenSearch with inverted indexes; understand the Lucene segment model.

**Distributed-systems concepts the seniors are graded on:**

- **CAP theorem** is *partition-time* trade-off only. The full picture is **PACELC**: during a Partition choose A or C; Else (normal operation) choose between Latency and Consistency. Cassandra is **PA/EL**; ACID databases are **PC/EC**; DynamoDB defaults to **PA/EL**.
- **Consistent hashing** with virtual nodes — solve hot-shard and rebalance cost. Arpit Bhayani's masterclass explicitly covers "operational complexities of consistent hashing" in Week 4.
- **Consensus algorithms:** Raft (understandable, leader-based, used by etcd, Consul, TiKV) vs Paxos (the original, Multi-Paxos optimization, used by Spanner, Chubby). Strong candidates know that *informal* leader election leads to split-brain — Google's SRE book is explicit about this.
- **Vector clocks** (Dynamo) for causality; **Lamport clocks** for total ordering; **TrueTime** (Spanner) for external consistency via GPS+atomic clocks. Per Corbett et al., *"Spanner: Google's Globally-Distributed Database"* (OSDI 2012): *"Because Google controls the network environment for TrueTime masters and daemons, in practice the uncertainty interval varies between 1ms and 7ms."*
- **Saga pattern** for distributed transactions (orchestration vs choreography); **CQRS** (separate read/write models); **Event Sourcing** (append-only log of facts).
- **Quorum math:** R + W > N for strong consistency; Raft uses ⌊N/2⌋+1 for both.

**Database internals — the senior bar:**

- **B+ Trees** (Postgres, MySQL InnoDB): self-balancing, optimized for range scans, in-place updates, 100s of keys per node. Read-optimized.
- **LSM Trees** (RocksDB, Cassandra, ScyllaDB, LevelDB, HBase): memtable (skip list) + Write-Ahead Log → flushed to immutable SSTables → background compaction. **Write-optimized via sequential I/O.** Per Kleppmann's *Designing Data-Intensive Applications* (O'Reilly, 2017) Chapter 3, and corroborated by community benchmarks: on HDDs sequential writes are typically ~50–100× faster than random writes; on SSDs the gap narrows to roughly ~5–10× but remains significant. This is the throughput advantage that justifies LSM's extra read amplification.
- **WAL** (Write-Ahead Log): durability primitive; every change written to log before applying to data structure. Crash recovery replays the log.
- **MVCC** (Postgres, CockroachDB, InnoDB, CouchDB, LMDB): each write creates a new version; readers see a consistent snapshot without blocking writers. Postgres uses tuple visibility maps; LMDB uses copy-on-write B+ trees with a meta-page.
- **Sharding strategies:** range-based (hot-spot risk on sequential keys), hash-based (loses range queries), directory-based (lookup service, flexibility at cost of an extra hop), geo-based (Uber-style).

### 5. Company-Specific Interview Formats

**FAANG general shape (4–6 rounds, 45–60 min each):**

- **Google:** L3 candidates typically get *no* system design round; L4 gets one; L5+ gets one or two plus Googleyness. Heavy on first-principles reasoning, light on "name the AWS service." Coding round bar is high.
- **Meta:** 2 coding + 1 system design + 1 behavioral for E4; E5+ adds a product architecture round (data modeling, API design). The behavioral round specifically probes "moving fast and building with impact."
- **Amazon:** 4–5 rounds, every round laced with Leadership Principles. SDE-2+ gets one or two system design rounds; SDE-3+ adds bar-raiser. Prepare ≥10 LP stories.
- **Netflix:** smaller loop, very senior-focused, culture-deck-driven. Less algorithm grind; more architecture and judgment.
- **Apple:** team-specific; the loop varies enormously between teams. Hardware-software integration and privacy are common cross-cutting themes.

**Senior bar (L5/E5/SDE-2 — concretely):**
- *Strong hire signals:* "I'd choose Cassandra over DynamoDB because our write pattern is append-heavy and we need tunable consistency per query." Going 2–3 levels deep on the hardest component without being asked. Naming failure modes proactively: "If the primary DB goes down, read replicas can be promoted, but we lose the last few seconds of writes if we're using async replication."
- *Reject signals:* correct architecture but can't explain the reasoning; waiting for the interviewer to ask follow-ups; missing the cost dimension.

**Staff bar (L6+):** you drive the conversation entirely. Asking "what should I focus on?" is itself the failure signal. Discuss build-vs-buy, organizational impact, and design for extensibility ("I'm structuring the API this way so when we inevitably add feature X, the change is additive").

**AI Labs — Anthropic, OpenAI, DeepMind:**

Anthropic's system design round is **50–55 minutes**, often LLM-infrastructure-flavored. Reported questions: design an inference batching system (GPU processes 100 inputs/batch, users submit synchronously and wait — async-to-sync mapping is the crux); design Claude chat service; design a file distribution system across thousands of machines with bandwidth constraints; design a token-generation service handling 100,000 RPS. The on-site is **two separate loops on different days**: Loop 1 = system design + coding + culture fit; Loop 2 = experiences & goals + technical project deep-dive. If you fail Loop 1, Loop 2 is cancelled. **Trick to internalize:** abstract every AI-specific framing into a generic distributed-systems primitive — "batch inference on a GPU" becomes "batched processing on a constrained compute resource." The patterns (queuing, async, GPU resource management, safety guardrails, abuse detection) repeat across all Anthropic questions.

OpenAI's system design round leans similar — infrastructure for inference, evaluation pipelines, multi-tenancy. The coding bar is high but not LeetCode-Hard-heavy; cleanliness matters.

**SaaS — Stripe (the unique one):**

Stripe runs a **5-round onsite**: two coding rounds, the signature **API Design Round** (unique to Stripe — design a developer-facing API with idempotency, versioning, error semantics, pagination, multi-currency), a **system design round** (often ledger or webhook delivery), and a behavioral round. There's also an **Integration Round** where you work against a real or realistic Stripe-shaped API with starter code and a small product task; this tests how you read documentation under time pressure, not memory. Stripe's culture filter: **writing rigor** (they make decisions through written design docs, not slide decks) and **developer empathy** ("Does this make the developer's life easier?"). Common question: design a ledger service — interviewers probe **exactly-once semantics, duplicate detection, downstream-bank-timeout-but-committed scenarios**. Use Whimsical for whiteboarding. Five endpoints that compose beautifully beats thirty endpoints that don't.

**Data Engineering — Databricks, Snowflake, Confluent:**

Databricks runs **3 coding rounds + 1 system design + 1 behavioral**. The system design is conducted on **Google Docs** (no whiteboard) — this is jarring if you're not ready. Concurrency/multithreading is its own dedicated coding round (implement a thread-safe logger that processes a message queue). Expect Spark internals depth: Catalyst optimizer (parsed → analyzed → optimized logical plan → physical plan with join strategy selection — broadcast hash join, sort-merge, shuffle hash), Delta Lake ACID properties, lakehouse vs warehouse positioning vs Snowflake. The coding bar leans LeetCode-Hard, with Databricks-tagged questions in recent months drawn directly from interview pool. **References weigh heavily** in the final decision — typically 1 manager + 2 senior teammates.

Snowflake interviews emphasize traditional database internals (query optimization, columnar storage, vectorized execution). Both companies converge on multi-tenant SaaS distributed systems.

**Quant / Finance — Jane Street, Two Sigma, Citadel, Goldman:**

- **Jane Street (the OCaml shop)** — Jane Street's own careers page (janestreet.com/preparing-for-a-software-engineering-interview/) explicitly says: *"We don't ask software engineers to do mental math, or math olympiad questions, or to contemplate logic puzzles. SWE interviews are about programming, plain and simple… We also don't award bonus points for using a functional language like OCaml. Please don't use OCaml just because you think it will make us happy."* SWE phone screen = 1 coding question in a shared online editor; their classic "Memo" problem expects a hash-table memoization solution. Per Jane Street's engineering blog: *"To allow us to work on the question together, we'll use a shared online editor… A typical first solution we're looking for at this stage uses a hash-table to store calculated results."* Onsite = a full-day Super-Day with ~3–4 coding rounds (60 min + 10 min Q&A) and 1 behavioral, **two interviewers per round** per Exponent's aggregated reports. No traditional FAANG-style system design round for SWE.
- **Two Sigma** — OA on HackerRank/Codility, 75–180 min, LeetCode-medium-to-hard with emphasis on boundary handling and complexity optimization. Three 60-minute technical phone rounds followed by behavioral. Engineering-perspective focus: code organization, scalability, real production-shape problems. Two Sigma's official careers page explicitly forbids ChatGPT/AI use during exams.
- **Citadel / Citadel Securities** — Citadel Securities' official engineering interview process page (citadelsecurities.com/careers/career-perspectives/our-engineering-interview-process/) describes the official flow: *"The second-round usually consists of three 45-minute interviews. Like the first round, it assesses a combination of technical and behavioral skills… Following the second round interviews, hiring managers across teams within Citadel and Citadel Securities have the opportunity to review your resume and interview feedback to determine where you would fit best."* The interview is *deliberately team-agnostic* in early rounds. Candidate reports on Wall Street Oasis describe **deep C++ grilling**: *"Superday was 3x45 min interviews, which ranged from cpp grilling on iterators, memory management, and stl containers such as std::variant, std::vector… Know c++ in depth, review OS and comp arch."* Finance-specific system design: order books, real-time pricing engines, low-latency message processing.
- **Goldman Sachs** — Per Interview Kickstart's Glassdoor-cited research: *"You will have five to six interview rounds for software engineering positions at Goldman Sachs. Every round will last for 30 to 60 minutes. The entire Goldman Sachs interview process takes up to 54 days on average."* Stack: HackerRank OA (1 hard + 1 easy reported on Glassdoor) → CoderPad live coding → **Superday with 3 back-to-back rounds**: pure DSA + concurrency (multi-threading, locks, race conditions, BlockingQueue, ThreadPool patterns), DSA + Low-Level System Design (TinyURL, parking-lot-shaped problems per Glassdoor), and an experience-driven round. Preferred languages: Java, C/C++, Python, JavaScript. System design depth is "low-level" / object-oriented rather than distributed FAANG scale.

**Consulting Tech — McKinsey QuantumBlack / McKinsey Digital:**

QuantumBlack replaces the standard McKinsey Solve game with a **coding assessment**. Per Hacking the Case Interview: *"QuantumBlack interviews replace the Solve with a coding assessment and add Technical Experience Interviews… The QuantHub test is a multiple-choice assessment lasting 72 to 100 minutes. It has three sections with approximately 12 questions each, covering programming, statistics, and data modeling… The QuantumBlack interview process has four to six rounds spread across four to eight weeks."* Typical sequence per a Glassdoor candidate report: *"1st round is a hackerank technical assignment along with a data science quiz. 2nd round is a pair programming interview along with a technical experience interview. 3rd round is a classic McKinsey style case and a tech based case."* Avg hiring duration 43 days per Glassdoor; difficulty 3.35/5. No FAANG-style dedicated system design round; instead, McKinsey Digital SWE "cases" probe architecture trade-offs (scalability vs cost vs performance) embedded in a business framing — pair programming is collaborative/applied where clean code + communication matters more than LeetCode tricks.

### 6. SDK / Library / Package Building

**LangGraph — what it actually is, architecturally.**

LangGraph models agent workflows as **directed graphs** built on the **BSP / Pregel algorithm** (the same algorithm Google used for their planet-scale graph computing system). The team explicitly chose Pregel over topological-sort DAG algorithms because **agents need cycles** (a topological sort cannot loop). Per the official LangChain blog "Building LangGraph": *"We ended up building on top of the BSP/Pregel algorithm, because it provides deterministic concurrency, with full support for loops (cycles)."* Core primitives:

- **State:** typed shared data structure (TypedDict / dataclass / Pydantic BaseModel) flowing through the graph.
- **Nodes:** Python functions that receive State, do work (LLM call, tool call, or plain Python), return a State update.
- **Edges:** functions deciding the next Node — static or conditional.
- **Channels:** named, versioned containers for State data; Nodes subscribe to one or more channels and run when channels change. Nodes inactive by default; activate on incoming message.
- **Super-steps:** discrete iterations of the graph; parallel Nodes are in the same super-step, sequential Nodes in separate super-steps.

The architecture exists to deliver six features the LangChain team identified as essential for production agents: **parallelization, streaming, checkpointing, human-in-the-loop, tracing, and task queue** (the task-queue piece is in LangGraph Platform, not the OSS library — that's an explicit boundary). The `Send` primitive enables map-reduce patterns; the `Command` primitive lets nodes both update state and route execution.

**LangChain — the package layout.** Worth studying as a model for *how to structure a Python library that integrates many providers*:

- `langchain-core`: base abstractions (Runnable, ChatModel, prompts, messages, output parsers) — zero provider dependencies.
- `langchain` (the meta-package): chains, agents, retrieval strategies — the cognitive architecture layer.
- `langchain-community`: third-party integrations, community-maintained.
- Integration packages: `langchain-openai`, `langchain-anthropic`, `langchain-google`, `langchain-redis` — lightweight adapter packages, each co-maintained with the integration developer.
- `langchain-experimental`: research-grade code.
- `langgraph`: stateful multi-actor orchestration (now the implementation foundation for LangChain's own agent abstraction — per docs.langchain.com: *"LangChain's agents are built on top of LangGraph"*).
- `langserve`: deploy Runnables as REST APIs.
- `langsmith-sdk`: independent, depends on nothing in langchain-core.

The **LCEL (LangChain Expression Language)** uses Python's `|` operator (`__or__`) for declarative composition: `prompt | model | output_parser`. This is the **Pipeline pattern + Builder pattern + functional composition** rolled into one operator override.

**OpenAI Python SDK — the gold-standard wrapper pattern.** Generated from OpenAPI spec by Stainless. Layered architecture you should copy when wrapping any REST API:

- `BaseClient` (abstract) — HTTP infrastructure, retry logic, error parsing.
- `SyncAPIClient` / `AsyncAPIClient` — concrete sync/async on top of `httpx.Client` / `httpx.AsyncClient`.
- `OpenAI` / `AsyncOpenAI` / `AzureOpenAI` — exposed client classes; API resources mounted as lazily-initialized properties (`client.chat.completions`, `client.fine_tuning.jobs`, `client.embeddings`).
- `with_options(...)` — returns a copy of the client with per-call overrides (timeout, http_client) without mutating shared state. This is the **Builder pattern** for request config.
- Dual type system: TypedDicts for request params (compile-time checks); Pydantic models for responses (runtime validation + autocomplete).
- Automatic retries on 408/409/429/5xx + connection errors. Default 2 retries. Idempotency-key support for safe replays.
- Default `httpx.Timeout` of 10 minutes; configurable per-client and per-request.
- Custom `http_client` injection lets users plug proxies, transports, or custom TLS — example: `OpenAI(http_client=DefaultHttpxClient(proxy=..., transport=httpx.HTTPTransport(local_address="0.0.0.0")))`.
- Error hierarchy: `APIConnectionError`, `RateLimitError` (429), `APIStatusError`, granular subclasses (`BadRequestError`, `AuthenticationError`, `PermissionDeniedError`, `NotFoundError`, `UnprocessableEntityError`, `InternalServerError`).
- Response objects expose `_request_id` (public despite the underscore) for support tickets.

**Anthropic SDK** mirrors this almost feature-for-feature (also Stainless-generated). When you build your own wrapper SDK, copy this pattern wholesale.

**Plugin architectures.** The dominant Python patterns:
- **Entry points** via `pyproject.toml` `[project.entry-points."myapp.plugins"]` — used by pytest, black, jupyter.
- **Registration decorators** — `@register("name") def my_impl(...)` populating a module-level dict.
- **Protocol/ABC subclassing** — explicit interface, found at import time.

**API design for libraries:**
- **Fluent interface / Builder pattern:** `Query.filter(...).order_by(...).limit(10).all()` — each method returns `self` (or a new instance for immutability).
- **Hide construction complexity** behind factory methods or `from_config()` classmethods.
- **Use `__init_subclass__` or registries** for plugin extension points instead of forcing inheritance.

**Packaging — the modern Python (2026) checklist:**
- **Build backend:** Hatchling, setuptools, or maturin (for Rust extensions like polars).
- **Dependency manager:** `uv` (fastest, becoming standard) or Poetry; both write to `pyproject.toml`.
- **`pyproject.toml` (PEP 621)** is the single source of truth for metadata, dependencies, and build config — no more `setup.py`.
- **`src/` layout** with `src/my_package/__init__.py` exposing top-level imports via `__all__` (the OpenAI SDK uses this exact pattern to expose `OpenAI`, types, exceptions, and constants).
- **Versioning:** PEP 440 (Python's standard) is *almost* SemVer but with subtle differences. PEP 440 prereleases use `1.0.0a1` / `1.0.0b1` / `1.0.0rc1` (no hyphen). SemVer prereleases use `1.0.0-alpha.1`. **For libraries, use three-part SemVer-aligned versions**: bump MAJOR on breaking changes, MINOR on backward-compatible features, PATCH on bug fixes. Use `0.x.y` for unstable APIs.
- **Dependency specifiers:** for libraries, be *permissive* (`requests>=2.28`); for applications, pin (`requests>=2.28.0,<3.0.0`); for reproducible builds, lock files (`uv.lock` / `poetry.lock` / `requirements.txt`).
- **Tilde operator:** `~=1.2.3` ≡ `>=1.2.3,<1.3.0` (compatible release).
- **PyPI publishing:** `uv publish` or `twine upload`. Version numbers are immutable once published — you cannot re-upload.

**FastAPI as the HTTP foundation:** when your SDK needs both a client *and* a server-mode (LangServe pattern, OpenAI Agents SDK pattern), FastAPI gives you Pydantic-validated request/response types, automatic OpenAPI docs, and async via `httpx` underneath. The OpenAI Agents SDK explicitly recommends this layering — `openai-agents` for the agent loop + FastAPI for the HTTP surface + httpx underneath for streaming.

### 7. Cross-Cutting CS — How OS, Network, DB, and DS Show Up in System Design

**OS concepts:**
- **Process vs thread:** process = isolated memory, expensive context switch (~microseconds); thread = shared memory, cheap switch but synchronization overhead. Senior signal: knowing when to spawn processes (Python GIL bypass via `multiprocessing`) vs threads (I/O-bound) vs async (high-fan-out I/O).
- **Memory hierarchy:** L1 cache (~1 ns) → L2 (~5 ns) → DRAM (~100 ns) → SSD (~100 µs) → spinning disk (~10 ms) → network (~1 ms intra-DC, ~150 ms cross-continent). Memorize these — they justify every caching decision.
- **Epoll / kqueue / io_uring** as the foundation of single-thread high-concurrency servers (Redis, Nginx, Node).

**Networking:**
- **TCP** = reliable, ordered, stream — payment systems, RPC. **UDP** = unreliable, unordered, packets — video, DNS, gaming, QUIC base.
- **HTTP/2** brings multiplexing (no head-of-line blocking on the application layer), header compression (HPACK), server push. **HTTP/3** runs on QUIC (UDP-based, faster connection setup, no TCP head-of-line blocking at transport layer).
- **gRPC** = HTTP/2 + Protobuf + bidirectional streaming, ideal for internal microservices.
- **WebSockets** for persistent bidirectional channels (chat, real-time dashboards) — Arpit's masterclass dedicates a session to scaling them.

**Database theory:**
- **ACID** (Atomicity, Consistency, Isolation, Durability) — single-node SQL guarantees.
- **BASE** (Basically Available, Soft state, Eventual consistency) — NoSQL relaxation.
- **Isolation levels:** Read Uncommitted → Read Committed → Repeatable Read → Serializable. Postgres default is Read Committed; understand phantom reads, write skew, and how MVCC eliminates most read locks.
- **Normalization** (1NF/2NF/3NF/BCNF) for OLTP; denormalization for OLAP and read-heavy services.

**Data structures that show up in system design:**
- **Bloom filter:** probabilistic set membership, no false negatives, tunable false-positive rate; used in Cassandra/RocksDB to skip SSTable reads, BigTable to skip blocks, browsers to check malicious URLs. Memory-efficient at ~10 bits per element for 1% FPR.
- **Count-min sketch / HyperLogLog:** approximate cardinality and frequency — Redis HLL, Twitter stream analytics.
- **Skip list:** Redis sorted sets, RocksDB memtable — O(log n) ordered ops without rebalancing.
- **Consistent hashing ring with virtual nodes:** Cassandra, DynamoDB, Memcached cluster. Without virtual nodes, removing a node causes hot-spots.
- **Merkle trees:** anti-entropy in Dynamo/Cassandra, Git, IPFS.
- **Trie / inverted index:** autocomplete, search engines.

---

## Details — A Recommended 12-Week Senior Study Plan

| Weeks | Track | Resources |
|---|---|---|
| 1–2 | OOP + SOLID + Patterns | Refactoring.Guru patterns, NLog/Django case studies, write 10 Python pattern examples |
| 3–4 | LLD problem grind | `awesome-low-level-design`, Hello Interview LLD section, Coudo AI; solve 10 problems |
| 5–8 | HLD + Distributed systems | **Arpit Bhayani's Masterclass** (recordings track if outside live cohort), DDIA book in parallel |
| 9 | Database internals | Arpit's Redis Internals course + RocksDB blog posts + Postgres MVCC papers |
| 10 | SDK/Library building | Read openai-python source, build a small REST wrapper SDK, publish to TestPyPI |
| 11 | Capacity estimation + mock interviews | Hello Interview, IGotAnOffer, Exponent |
| 12 | Company-specific deep prep | Engineering blogs of your target (Stripe blog for Stripe; Databricks engineering blog for Databricks) |

---

## Recommendations

**If your goal is a senior FAANG role (L5/E5/SDE-2):**
1. Pay for Arpit Bhayani's Masterclass recordings (₹39,999) — it is the highest-density distributed-systems-and-implementation resource available; the 1:1 mentorship slot alone is worth a chunk of the cost.
2. In parallel, grind 20 LLD problems from `awesome-low-level-design`, writing complete code for at least 8 of them.
3. Read DDIA (Designing Data-Intensive Applications) cover-to-cover.
4. Do 5+ mock HLD interviews (Hello Interview / IGotAnOffer / interviewing.io).

**If your goal is an AI lab (Anthropic, OpenAI):**
1. Master the **inference-batching question** specifically — it covers ~70% of the patterns Anthropic asks. Practice the async-to-sync mapping explicitly.
2. Read the LangGraph "Building LangGraph" blog post and the OpenAI Python SDK source on GitHub — this is the architectural vocabulary they expect.
3. Prepare a 3–4-level-deep "ethical conflict at work" story for the culture round.

**If your goal is Stripe:**
1. Build a small idempotent REST API yourself end-to-end (Stripe-shaped: charges, refunds, webhooks). The Integration Round is observation, not memorization.
2. Practice the **5-endpoint discipline** — design fewer, more composable endpoints rather than many granular ones.
3. Write a one-page design doc for a recent system you built and get peer feedback.

**If your goal is Databricks / Snowflake:**
1. Read three Databricks engineering blog posts deeply (Photon, Delta, Unity Catalog). These map directly to interview questions.
2. Drill Spark internals (Catalyst, Tungsten, AQE) until you can whiteboard the query plan flow.
3. Practice system design on Google Docs — type, don't draw. This is a surprisingly different mode.

**If your goal is quant/finance (Citadel, Jane Street, Two Sigma):**
1. For Citadel: a C++ deep-dive on STL, move semantics, memory management; review OS and computer architecture trivia.
2. For Jane Street: stop fearing OCaml — Jane Street's own page tells you to use Python or your strongest language; the bar is *programming* clarity, not language choice.
3. For Two Sigma: HackerRank-style boundary-handling and complexity optimization under 75–180 min time pressure.

**Benchmarks that should change your plan:**
- If you cannot complete a parking-lot LLD in 45 minutes with clean SOLID application: spend another 2 weeks on LLD before HLD.
- If your back-of-envelope numbers are within 1 order of magnitude of ByteByteGo's Twitter example: you're ready for capacity estimation.
- If you can articulate PACELC for 3 different databases without notes: you're ready for distributed-systems trade-off questions.
- If you have not built a single open-source SDK: do not skip the SDK chapter — Stripe and AI labs both probe this.

---

## Caveats

- **Course pricing and cohort dates change.** ₹49,999 is the June 2026 cohort number from arpitbhayani.me at the time of research; verify before paying.
- **AI labs' question banks expand quickly.** Anthropic specifically had a small set of system design questions a year ago that is now broadening per recent candidate reports. Don't drill only the inference-batching question.
- **Some sources for company interview structure are aggregators** (Exponent, Interview Query, AlgoMonster, OphyAI). They synthesize candidate reports, which can conflict on round counts. Where official career pages exist (Jane Street, Citadel Securities), trust those over aggregators. For Goldman, the Glassdoor-based ~54-day average and 5–6-round count are aggregate statistics, not a Goldman-published figure.
- **The Jane Street onsite round count conflicts even within one source** — Exponent describes both 3 coding + 1 behavioral and 4 coding rounds in the same article. Treat as "≈3–4 coding rounds + 1 behavioral." Jane Street's own page does not publish a specific count.
- **The Citadel content split (50/25/15/10)** cited in some guides is an *aggregator estimate*, not firm-published data. Use it as rough guidance only.
- **YouTube/Twitter follower counts vary by source and date** — Arpit's own website cites different numbers across pages, so subscriber figures in this guide are not cited as precise.
- **LangGraph / LangChain APIs change between versions.** The architectural primitives described (State, Nodes, Edges, channels, Pregel) are stable, but specific class names and import paths shift; check current docs before coding.
- **Use of AI tools during interviews is explicitly forbidden** at Two Sigma and increasingly elsewhere. Several "interview copilot" services aggressively marketed in 2026 search results (Linkjob, Ophyai, Phantom Code) are operating in a gray zone; using them risks immediate disqualification and is unethical. This guide treats them as data points about *what topics get asked*, not as endorsed prep tools.
- **LSM-vs-B-tree write speedup figures** (50–100× HDD, 5–10× SSD) are textbook estimates from Kleppmann's DDIA Chapter 3 and community benchmarks; the exact ratio depends heavily on workload shape, fsync policy, and compaction settings — treat as order-of-magnitude only.
- **Generative-AI-augmented coding assessments are evolving.** Some firms now allow AI tools in OAs while explicitly forbidding them in interviews; read every recruiter email carefully.