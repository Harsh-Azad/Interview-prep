# The Forward Deployed Engineer Study Roadmap — DevOps · MLOps · LLMOps

*Tailored for Satvik — Bengaluru-based technical founder, TypeScript/Next.js/tRPC/Prisma/BullMQ/Vercel-AI-SDK background, GEO Code (Shopify) + EY AI tax assistants, targeting AI-lab FDE / FAANG SRE / Data-platform SA / Quant-infra roles.*

---

## TL;DR

- **The FDE role is a "T-shaped systems engineer who can also negotiate, ship inside a customer VPC, and debug an LLM pipeline at 2 a.m."** Anthropic, OpenAI, Palantir, Scale AI, Databricks, and Google Cloud are all hiring this title for the same archetype: production code + customer-facing judgement + frontier-LLM literacy. Your founder + Vercel-AI-SDK + EY-deployment story already lines up; the four real gaps are (a) **production Python + MLOps depth**, (b) **inference internals fluency** (vLLM/FlashAttention/KV cache), (c) **Kubernetes + observability + IaC** at production rigor, and (d) the **"decomposition / ambiguous-client scenario"** interview muscle.
- **Plan ~16 weeks in three arcs**: Weeks 1–4 fix DevOps/Linux/K8s/cloud + Python foundations; Weeks 5–10 own MLOps + LLMOps internals (distributed training, RAG, vLLM, evals); Weeks 11–16 build FDE-grade portfolio work (an MCP server + agent for a real customer-shaped workflow, deployed in a VPC pattern, with full observability + an eval harness) and run mock loops in the format each target company actually uses.
- **Bias your prep**: 40% systems & production debugging, 30% LLM/Agent internals, 20% customer-scenario decomposition + system design, 10% LeetCode-medium Python/SQL. Do *not* over-index on LeetCode-hard — every FDE-role loop (Palantir per fde.academy, OpenAI per Gaijineer's firsthand, Anthropic per interviewing.io) explicitly de-emphasizes algorithm puzzles in favor of practical coding + ambiguity handling.

---

## Key Findings (the things that should change how you prep)

1. **The FDE role is structurally distinct from a SWE role.** Anthropic's official JD: *"Work within customer systems to build production applications with Claude models … Deliver technical artifacts for customers like MCP servers, sub-agents, and agent skills that will be used in production workflows."* OpenAI's official JD: *"Forward Deployed Engineers (FDEs) lead complex end-to-end deployments of frontier models in production … You will own discovery, technical scoping, system design, build, and production rollout."* Both expect 25–50% travel and demand customer-facing fluency **and** production code.
2. **The market just inflected.** On May 11, 2026, OpenAI formalized "The Deployment Company" — a joint venture majority-owned by OpenAI that raised more than $4B from exactly 19 firms, led by TPG with Advent International, Bain Capital, and Brookfield Asset Management as co-lead founding partners; consulting/SI partners include Bain & Company, Capgemini, McKinsey & Company. The day after, Google Cloud CEO Thomas Kurian publicly recruited for FDE on LinkedIn; 59 roles were posted in week one across the US, India, Brazil, Australia, Mexico, Singapore, South Korea, and Canada, with US base salary bands from $127K–$183K (Applied FDE) up to $183K–$265K (FDE IV).
3. **The "deployment scenario" round is decisive at every AI-lab FDE.** Palantir's loop centers on an open-ended 30–60-min ambiguous-client scenario; per fde.academy: *"Jumping immediately to a solution is the most common failure point."* OpenAI uses an analogous ~5-hour take-home + customer role-play (Gaijineer firsthand).
4. **The competitive dynamics behind hiring**: per Menlo Ventures' 2025 Mid-Year LLM Market Update (n=150 technical leaders), *"Anthropic is now the leading enterprise LLM provider with 32% market share. OpenAI holds 25%, down from 50% in 2023. Google has taken third place, claiming 20% of enterprise usage."* And per MIT Media Lab Project NANDA's *The GenAI Divide: State of AI in Business 2025*: *"Just 5% of integrated AI pilots are extracting millions in value, while the vast majority remain stuck with no measurable P&L impact"* (i.e., 95% have no measurable impact). The FDE role exists to close that 95% gap — they are not closing it via models, they are closing it via deployment engineering. That's your value prop.
5. **For your TypeScript/Next.js background, the single highest-leverage skill gap is production Python + Linux + K8s.** Every AI-lab FDE listing requires Python (Anthropic: *"proficiency in Python (and ideally in one or more additional languages like TypeScript, Java)"*). Your TS stack is an asset but Python fluency is non-negotiable.
6. **LLMOps depth is what separates "GenAI consultant" from "FDE."** Knowing *why* vLLM is faster than HF Transformers (PagedAttention + continuous batching), *why* FlashAttention-3 hits 75% of H100 peak (per Dao et al.: *"FP16 reaching up to 740 TFLOPs/s (75% utilization)"*), *why* DPO replaced PPO for most preference tuning, and *how* speculative-decoding acceptance rate α determines actual speedup is the entry ticket to Anthropic/OpenAI loops.

---

## Details — The Full Roadmap

The roadmap runs in **four phases** (Foundation → Intermediate → Industry-Expert → FDE-Specific) over 16 weeks, with deep-dive content for each topic, **grill-down chains** for the major topics, canonical resources, and a **company-by-company emphasis matrix** at the end. Each topic block has: **Concept** (plain English) · **Why FDE cares** (the production failure mode) · **Grill-down chain** (L1 → L5 interview ladder) · **Cross-links** (other CS concepts) · **Resources**.

---

# PHASE 1 — FOUNDATION (Weeks 1–4)

Goal: re-establish college-grade fundamentals at production rigor. If you can already explain `epoll` vs `io_uring`, BGP, and Postgres MVCC, skim this phase in 1 week.

## 1.1 Linux Internals

**Concept.** Processes, threads, file descriptors, virtual memory, the page cache, `/proc`, signals, cgroups (v1 vs v2), namespaces (PID, NET, MNT, IPC, UTS, USER), systemd units/targets, boot sequence, syscalls.

**Why FDE cares.** When a customer's container is OOM-killed, when latency is high because the page cache is being evicted by a noisy neighbor, when a sidecar can't see the parent's mount namespace — only Linux fluency saves you.

**Grill-down (Linux/cgroups)**
- L1: "Process vs thread?"
- L2: "What does `clone()` do, and how does it differ from `fork()`?"
- L3: "Walk me through how Docker uses namespaces + cgroups — what's shared with the host?"
- L4: "Cgroups v2 unified hierarchy — what changed from v1, and why does this matter for Kubernetes memory accounting?"
- L5: "A pod is OOM-killed but `kubectl top` shows it under its limit. Diagnose this — include how the kernel computes memory pressure, the role of `memory.high` vs `memory.max`, and how page-cache accounting interacts with WSS."

**Cross-links.** OS memory management; container internals; observability (cAdvisor reads `/sys/fs/cgroup`).
**Resources.** *The Linux Programming Interface* (Kerrisk); Brendan Gregg's *Systems Performance, 2nd ed.*; `man 7 namespaces`, `man 7 cgroups`.

## 1.2 Networking Deep Dive

**Concept.** OSI/TCP-IP, TCP handshake + congestion control (Cubic, BBR), HTTP/1.1 keep-alive, HTTP/2 multiplexing + HPACK + HoL at TCP layer, HTTP/3/QUIC, gRPC framing (HTTP/2 + protobuf), DNS chain, CDN architecture (edge POPs, anycast), BGP basics, TLS 1.3 (0-RTT, ECH), L4 vs L7 LBs (NLB vs ALB/Envoy/NGINX), service mesh sidecar pattern.

**Grill-down (TLS)**
- L1: "Walk me through a TLS handshake."
- L2: "What did TLS 1.3 remove from 1.2, and why?"
- L3: "Why does 0-RTT have replay-attack risk? When is it safe?"
- L4: "Customer's LB terminates TLS and re-encrypts to backend with a different cipher. Where do you put PII redaction in this path?"
- L5: "Design mTLS for an air-gapped on-prem deployment of your LLM inference service across 3 customer-owned racks — cert rotation, CRL distribution, recovery from CA compromise."

**Cross-links.** Security (zero trust, OAuth/OIDC, SPIFFE/SPIRE); distributed systems (gRPC + protobuf is the backbone of ML serving — Triton, KServe).
**Resources.** *Computer Networking: A Top-Down Approach*; Cloudflare's blog on QUIC; *High Performance Browser Networking* (Ilya Grigorik).

## 1.3 Data Structures & Algorithms — The Subset That Matters

FDE interviews almost never test LeetCode-hard. They *do* test:
- **HashMap / dict** internals (open addressing vs chaining)
- **Heap / priority queue** (HNSW search frontier; schedulers)
- **Trie** (autocomplete, prompt routing)
- **Bloom filter** (dedup at ingest)
- **HyperLogLog** (unique-event cardinality with bounded memory)
- **MinHash / LSH** (near-dup detection in training-data dedup)
- **Skip list** (foundation for HNSW intuition)
- **B-tree vs LSM-tree** (read- vs write-optimized DBs)
- **Reservoir sampling** (sampling streams with bounded memory)
- **Top-K with quickselect / heap** (sampling logits)

**Grill-down.** "Dedupe 50B web docs with 1TB RAM" → MinHash + LSH → "What error rate?" → "Parallelize how?" → MapReduce + bucketed shuffle.
**Resources.** *Designing Data-Intensive Applications* (DDIA) Ch. 3 LSM vs B-trees is canonical.

## 1.4 Databases & Storage

**Concept.** Relational vs document vs columnar (Parquet/ORC) vs object stores; ACID; isolation levels (RC, RR, serializable, snapshot isolation); MVCC (Postgres + Snowflake); B-trees vs LSM (Cassandra, RocksDB); WAL; replication (sync vs async); sharding; query optimizers; vector indexes — HNSW, IVF, IVF-PQ, ScaNN, DiskANN.

**Grill-down (HNSW)**
- L1: "What's an HNSW?"
- L2: "Why hierarchical layers — analogy to a skip list?"
- L3: "Walk through insertion: how is the layer for a new node decided, what's `efConstruction`, what's `M`?" *(Layer assignment is an exponentially decaying probability; M caps neighbor count; efConstruction is candidate-list size during build.)*
- L4: "HNSW vs IVF-PQ for 1B vectors on a single box?" *(HNSW: in-memory, high recall, expensive updates; IVF-PQ: disk-friendly, compressed, lower recall, faster build.)*
- L5: "Filtered vector search (e.g., 'similar to X but only for user-org=42') — pre-filter vs post-filter vs Weaviate/Qdrant 'filterable HNSW' tradeoffs in latency and recall?"

**Cross-links.** HNSW is graph + skip list. IVF is k-means as a coarse quantizer. PQ is vector quantization (Jégou et al. 2010).
**Resources.** DDIA; Malkov & Yashunin (2016) HNSW paper; FAISS docs.

## 1.5 Operating Systems — Production Subset

Scheduling (CFS, real-time vs normal); virtual memory, demand paging, OOM killer; I/O models — blocking, non-blocking, async (`epoll`, `io_uring`); `mmap` vs read/write; `fsync()` and `O_DIRECT`.

**Grill-down.** "Batch ingest is slow on 96 vCPU but CPU is idle." → iowait → blocked on disk → `iostat`/`iotop`/`blktrace` → small random IO → tune to large sequential or NVMe.

## 1.6 System Design Fundamentals

CAP, PACELC, consistency models (linearizable, sequential, causal, eventual), consensus (Paxos, **Raft** — know it cold: leader election, log replication, safety, snapshotting), 2PC vs Saga, idempotency keys, exactly-once vs at-least-once, backpressure, circuit breakers, bulkheads, rate limiting (token bucket, leaky bucket), load shedding.

**Grill-down (Idempotency)**
- L1: "Why idempotency keys for payments / LLM retries?"
- L2: "Where do you store the key — DB? Redis? For how long?"
- L3: "What if the key is stored but the operation is still in-flight when the retry arrives?"
- L4: "Design idempotent LLM inference where the same key returns the same completion even with `temperature > 0`." *(Seed the sampling; cache the completion against the key with a TTL aligned to the customer's exactly-once window.)*

**Resources.** DDIA; *Site Reliability Engineering* (free at sre.google); Alex Xu *System Design Interview Vol 1+2*; Martin Kleppmann's distributed-systems lecture series.

---

# PHASE 2 — DEVOPS PRACTITIONER DEPTH (Weeks 3–6, overlaps with Phase 1)

## 2.1 Containers — Docker Internals

Docker CLI is the user-facing wrapper; `containerd` is the high-level runtime; `runc` is the low-level OCI runtime that actually does the kernel work — `clone()`/`unshare()`/`pivot_root()`. Images are content-addressable layer stacks; OverlayFS gives copy-on-write.

**Grill-down (Docker)**
- L1: "What is Docker?"
- L2: "Walk through `docker run nginx`."
- L3: "What's in an image layer; why content-addressed and immutable?"
- L4: "`docker build` is rebuilding a cached layer. Why?" *(COPY/ADD mtime or content changed; parent layer changed; `--no-cache`; BuildKit cache mounts not configured.)*
- L5: "Explain `docker run` → containerd → runc → kernel at the syscall level." *(runc creates the namespace set via `unshare`/`clone` flags, sets cgroups, applies seccomp + AppArmor, `pivot_root` into rootfs, `execve` into entrypoint.)*

## 2.2 Kubernetes — From `kubectl apply` to the Kernel

**Control plane**: kube-apiserver (only writer to etcd), etcd (Raft KV), kube-scheduler, kube-controller-manager, cloud-controller-manager.
**Node components**: kubelet, kube-proxy, CNI plugin, container runtime (CRI).
**Workload primitives**: Pod, ReplicaSet, Deployment, StatefulSet, DaemonSet, Job, CronJob.
**Networking**: ClusterIP/NodePort/LoadBalancer/Ingress; CNI (Calico, Cilium); NetworkPolicy; service mesh (Istio, Linkerd, Cilium mesh).
**Storage**: PV/PVC, StorageClass, CSI drivers.
**Scaling**: HPA, VPA, Cluster Autoscaler, KEDA (event-driven).
**Extensibility**: CRDs + operators (watch → reconcile → act).
**Security**: RBAC, ServiceAccounts, PodSecurity admission (replaced PSP), NetworkPolicies, Secrets (base64, not encrypted), Pod Security Standards.

**Grill-down (canonical K8s ladder)**
- L1: "What is Kubernetes?"
- L2: "How does the scheduler work?" *(Two-phase: filtering — predicates like resource availability, taints/tolerations, node affinity — then scoring — least-allocated, image locality, inter-pod affinity. Then bind.)*
- L3: "Walk me through `kubectl apply -f deployment.yaml`." *(kubectl POSTs YAML to kube-apiserver → admission controllers run → object written to etcd → Deployment controller observes via watch → creates ReplicaSet → ReplicaSet controller creates N unscheduled Pods → scheduler assigns nodes via `spec.nodeName` → target node's kubelet observes assignment, calls CRI to pull image, sets up cgroups + namespaces, starts container → kubelet reports status → etcd updated → `Running`.)*
- L4: "Debug a pod stuck in CrashLoopBackOff." *(`kubectl describe pod` for events; `kubectl logs --previous`; check liveness probe; image entrypoint; exec into init container; resource limits — OOM at startup?; `kubectl get events --field-selector involvedObject.name=…`)*
- L5: "How does kubelet talk to the container runtime — explain the CRI." *(Per Kubernetes docs: "the Container Runtime Interface (CRI) is the main protocol for the communication between the kubelet and Container Runtime." It's gRPC with two services — `RuntimeService` (RunPodSandbox, CreateContainer, StartContainer…) and `ImageService` (pull/list/remove). Kubelet is the gRPC client; containerd/CRI-O expose the server on a Unix socket like `/run/containerd/containerd.sock`. K8s 1.24 removed the dockershim.)*

**Cross-links.** etcd → Raft; controller pattern → reactive loop / event sourcing.
**Resources.** *Kubernetes Up and Running*; *Programming Kubernetes*; k8s.io docs; Liz Rice "Container Security."

## 2.3 CI/CD & GitOps

GitHub Actions (matrix builds, reusable workflows, OIDC to AWS for keyless auth); Jenkins legacy; Tekton (K8s-native); ArgoCD vs Flux; Spinnaker. Deployment strategies: rolling, blue/green, canary, shadow; feature flags (LaunchDarkly, Unleash). Progressive delivery: Argo Rollouts, Flagger. Supply-chain security: **SLSA** levels 1–4, Sigstore/cosign, SBOMs (SPDX/CycloneDX).

**Grill-down.** "Canary at 5% shows a 0.3% error-rate bump. Roll back or forward?" → significance, error budget remaining, blast radius, time-to-rollback, customer impact.

## 2.4 Infrastructure as Code

- **Terraform**: HCL, providers, modules, the state file (and why it's the most dangerous file in your org), backends (S3 + DynamoDB lock), workspaces, `terraform import`, drift, `lifecycle { prevent_destroy }`.
- **Pulumi**: same primitives, real languages.
- **Crossplane**: K8s-native control-plane-as-code.
- **Helm**: templating + release management; Helm 3 (no Tiller).

**Grill-down (Terraform)**
- L1: "What is Terraform?"
- L2: "What's in the state file and why does it matter?"
- L3: "Two engineers run `apply` simultaneously. What happens, how do you prevent it?" *(State lock via DynamoDB / TF Cloud / Consul backend.)*
- L4: "State drifted because someone changed the AWS console. Reconcile."
- L5: "Design a TF module for an LLM-serving stack (VPC, EKS, GPU node group, S3 model store, KMS, IAM, IRSA, secrets) reusable across 3 customer VPCs with different compliance regimes."

## 2.5 Cloud Platforms — Comparative Map

| Capability | AWS | GCP | Azure |
|---|---|---|---|
| Managed K8s | EKS | GKE (autopilot) | AKS |
| Object store | S3 | GCS | Blob |
| Serverless | Lambda | Cloud Run / Functions | Functions |
| Data warehouse | Redshift | BigQuery | Synapse |
| Vector | OpenSearch k-NN, Aurora pgvector | Vertex AI Vector Search | AI Search |
| ML platform | SageMaker | Vertex AI | Azure ML |
| Secrets | Secrets Manager / Parameter Store | Secret Manager | Key Vault |
| Auth | IAM / IRSA | IAM / Workload Identity Federation | Entra ID / Managed Identity |

**FDE relevance.** Customers are on all three. Know IAM cold (role assumption, trust policies, IRSA for K8s SAs, GCP WIF), VPC + PrivateLink, and how Bedrock / Vertex AI Model Garden / Azure OpenAI provide enterprise-compliance wrappers around frontier models.

## 2.6 Observability — Three Pillars + The Methods

- **Metrics**: Prometheus (pull-based, PromQL), Cortex/Thanos/Mimir for HA, Grafana, OpenMetrics.
- **Logs**: ELK / Loki / Fluentd / Vector / OpenSearch.
- **Traces**: OpenTelemetry (now the standard), Jaeger / Tempo / Honeycomb.
- **Profiles** (fourth pillar): pprof, Parca, Pyroscope.
- **Methods**: RED (Rate, Errors, Duration — request-driven services); USE (Utilization, Saturation, Errors — resources); Four Golden Signals (latency, traffic, errors, saturation — Google).
- **SLI/SLO/SLA/Error Budget.** Per the Google SRE workbook: *"An error budget is 1 minus the SLO of the service. A 99.9% SLO service has a 0.1% error budget. If our service receives 1,000,000 requests in four weeks, a 99.9% availability SLO gives us a budget of 1,000 errors."* The SRE-book-recommended pattern is **multi-window, multi-burn-rate** alerting (defends both fast and slow burns; avoids alert fatigue).
- **Incident response**: on-call rotations (PagerDuty/Opsgenie/incident.io), severity levels, blameless postmortems, "5 Whys" + contributing factors.

**Grill-down (SLO design)**
- L1: "Difference between SLI, SLO, SLA?"
- L2: "How do you pick an SLO target?"
- L3: "99.9% latency SLO has been burning at 4× for 1 hour. What does multi-window-multi-burn-rate alerting do here that a static p99 alert doesn't?"
- L4: "Customer's LLM app has a p95 latency SLO of 3s. Reranking pushed p95 to 5s. Violate SLO or skip rerank? Frame in error-budget terms."
- L5: "Design SLIs for an LLM inference service. What's a 'good event' when responses are stochastic?" *(Structural correctness — valid JSON in JSON mode; TTFT; full-response latency; refusal-rate band; faithfulness against eval set. Separate SLO per axis.)*

## 2.7 Security — DevSecOps + AI Security

- **SAST**: Semgrep, CodeQL, Snyk Code. **DAST**: OWASP ZAP, Burp.
- **Supply chain**: SLSA levels 1–4, Sigstore, SBOM, dependency pinning, Dependabot/Renovate.
- **Secrets**: HashiCorp Vault (dynamic secrets, transit engine), AWS Secrets Manager, Sealed Secrets / SOPS for GitOps.
- **Zero Trust**: BeyondCorp; mTLS via SPIFFE/SPIRE; service identity over network identity.
- **OAuth 2.0 / OIDC / SAML / SCIM**: auth-code + PKCE; why JWTs need short TTLs.
- **Compliance**: SOC 2 Type II, ISO 27001, HIPAA, GDPR, FedRAMP (relevant to Anthropic's Federal Civilian FDE listing).
- **AI security**: prompt injection (direct + indirect), data poisoning, model extraction, membership inference, jailbreaks. **OWASP Top 10 for LLMs** is canonical. PII redaction. Differential privacy (ε-DP, Laplace mechanism).

## 2.8 SRE Principles — Beyond the Book

Toil reduction; capacity planning; chaos engineering (Chaos Monkey, Gremlin, Litmus); blameless postmortems; the Dickerson hierarchy: monitoring → incident response → postmortem → testing/release → capacity planning → development → product.

---

# PHASE 3 — MLOPS PRACTITIONER DEPTH (Weeks 5–9)

## 3.1 The ML Lifecycle

`Data ingest → Validation → Feature engineering → Training → Eval → Deploy (canary/shadow) → Monitor → Retrain trigger → loop`
**Google MLOps maturity model** (Practitioners' Guide to MLOps, 2021): Level 0 (manual, notebook-driven) → Level 1 (automated ML pipeline, continuous training) → Level 2 (CI/CD-automated pipeline changes themselves).

## 3.2 Data: Validation, Versioning, Feature Stores

- **Validation**: Great Expectations; TFDV; Deequ (Spark-native).
- **Versioning**: DVC; LakeFS; Pachyderm.
- **Feature stores**: Feast (OSS), Tecton (managed), SageMaker / Vertex AI native. Two concepts that *will* be grilled:
  - **Online vs offline store**: offline (Snowflake / BigQuery / S3-Parquet) for training; online (Redis / DynamoDB) for ms-latency serving.
  - **Point-in-time correctness**: when joining features to labels for training, use the feature value *as of* the event time, not the latest value — otherwise you leak future info and get train/serve skew at deploy.

**Grill-down (PIT correctness)**
- L1: "Why a feature store?"
- L2: "What is point-in-time correctness?"
- L3: "Show me an AS-OF / windowed left-join in Snowflake / BigQuery that does it correctly."
- L4: "Customer reports 0.3 AUC train/serve skew. Walk me through diagnosing." *(Compare feature distributions; check leakage; check online/offline computation symmetry; check timestamp handling; check imputation differences.)*

## 3.3 Training Infrastructure

### Distributed training mental model
- **DDP**: full model copy per GPU; gradient all-reduce after backward. Best when model fits on one GPU.
- **FSDP**: PyTorch-native ZeRO-3 equivalent. Shards params + grads + opt-states; gathers per-layer on use, re-shards. Sweet spot 100M–~10B for many setups.
- **DeepSpeed ZeRO**: Stage 1 shards opt-states; Stage 2 + grads; Stage 3 + params. ZeRO-Infinity adds CPU + NVMe offload for 100B+. Per ML Journey: *"FSDP FULL_SHARD and ZeRO Stage 3 are equivalent — both shard everything and scale linearly with GPU count."* DeepSpeed has more mature CPU/NVMe offload + fused Adam; FSDP has cleaner PyTorch integration.
- **Tensor Parallel** (Megatron-LM): split a matmul across GPUs. Essential at >10B.
- **Pipeline Parallel** (GPipe, PipeDream): split by layer; pipeline microbatches.
- **3D parallelism**: TP + PP + DP combined — what frontier-scale training uses.
- **Collectives** (NCCL): all-reduce (ring or tree), all-gather, reduce-scatter, broadcast.

### Hardware
- GPU memory hierarchy: HBM → L2 → SMEM → registers. FlashAttention's whole point is to avoid materializing the N×N attention matrix in HBM by tiling and keeping it in SMEM.
- **Hopper H100**: TMA, WGMMA, FP8. FlashAttention-3 exploits all three: per Dao et al., *"FP16 reaching up to 740 TFLOPs/s (75% utilization)"*; FP8 reaches ~1.2 PFLOPS.
- **MIG**: partition an A100/H100 into up to 7 isolated instances — useful for inference multi-tenancy.
- **GPUDirect / RDMA**: bypass CPU memory for GPU-NIC; InfiniBand SHARP for in-network reduction.

### Orchestration
- **Kubeflow** (K8s-native, KFP for pipelines, Katib HPO, KServe serving); **Argo Workflows** (DAG runner KFP is built on); **Ray** (actor model; Ray Train/Tune/Serve, RLlib); **SageMaker / Vertex AI / Azure ML**; **Metaflow** (Netflix); **Flyte** (Lyft); **Prefect**; **Dagster** (software-defined assets); **Airflow** (task DAGs).

**Grill-down (FSDP)**
- L1: "What is FSDP?"
- L2: "What does FULL_SHARD actually shard, and when does it gather?"
- L3: "Why is FSDP 10–20% slower than DDP for models that fit, and faster for models that don't?" *(Communication: param all-gathers per fwd/bwd.)*
- L4: "HSDP / ZeRO++ hpZ — what does hybrid sharding do?" *(Shard intra-node, replicate inter-node; cuts cross-node bandwidth.)*
- L5: "Fine-tuning 70B with FSDP across 16 H100s, NCCL all-reduce is the bottleneck. Diagnose and remediate." *(NIC topology — NVLink vs PCIe; RDMA?; flat ring vs hierarchical reduction; HSDP; bf16 reduction; gradient compression; `NCCL_DEBUG=INFO` + `nsys` profiling.)*

## 3.4 Experiment Tracking & Registries

MLflow (OSS, self-host), Weights & Biases (managed, best UI), Comet, Neptune. Track: params, metrics, artifacts, datasets, env (conda/pip lock), git SHA. Reproducibility = same hyperparams + same data + same seed + same code SHA → same model. Model registry: training run → registered model → stage (Staging/Prod/Archived) → deployment.

## 3.5 Hyperparameter Tuning

Optuna (TPE, CMA-ES, pruning); Ray Tune (ASHA, PBT, Hyperband); Vertex Vizier (managed Bayes).

## 3.6 Model Serving — Batch vs Online vs Streaming

- **Batch**: Spark, Beam, Airflow.
- **Online**: TF Serving, TorchServe, **NVIDIA Triton** (multi-framework, dynamic batching, model ensembles, instance groups), BentoML, KServe, Seldon, Ray Serve.
- **Triton specifics**: dynamic batching (`max_queue_delay_microseconds` is your latency knob); model ensembles (in-server pipelines); instance groups (multiple replicas per GPU); backends (PyTorch, ONNX, TensorRT, vLLM, Python BLS).
- **Patterns**: A/B test, canary (1% → 5% → 25% → 100%), shadow (mirror traffic, drop response — used to validate latency + correctness without user impact), blue/green.

## 3.7 Model Monitoring

- **Data drift**: input distribution shift. Metrics: KL divergence, **PSI** (Population Stability Index — <0.1 OK, 0.1–0.25 watch, >0.25 alert), KS statistic (continuous), Chi-square (categorical).
- **Concept drift**: P(Y|X) shifts.
- **Performance**: ground-truth (when labels arrive — accuracy, AUC, RMSE); proxies when labels are delayed.
- **Tools**: Evidently AI (OSS), Arize, WhyLabs, Fiddler, Aporia.

## 3.8 Compute Platforms — Lakehouse Architecture

- **Open table formats**: Delta Lake, **Apache Iceberg** (Netflix-origin, now de facto open standard — native in Snowflake, BigQuery, Trino, Spark, Flink), Apache Hudi. All give ACID + time travel + schema evolution + upserts on Parquet/ORC in object storage.
- **Databricks lakehouse**: Unity Catalog (governance), Delta Live Tables, Photon engine, MLflow.
- **Snowflake**: separation of storage (micro-partitions in cloud object store) and compute (virtual warehouses); Snowpark (Python/Scala); Cortex (in-warehouse LLM).
- **Spark**: narrow vs wide transformations, partitioning, broadcast joins, AQE (Adaptive Query Execution), shuffle.

## 3.9 ML Infrastructure Deep Cuts

GPU scheduling on K8s (NVIDIA device plugin; time-slicing for inference; MIG for hard partitions; KAI Scheduler for gang scheduling + fractional GPUs). **NCCL collectives**: ring all-reduce bandwidth = 2(N−1)/N × bus BW; tree better at very large N. Topology awareness: NVLink (intra-node), InfiniBand HDR/NDR (inter-node), RoCE v2 (Ethernet RDMA).

---

# PHASE 4 — LLMOPS DEPTH (Weeks 6–12)

## 4.1 Mental Model: The Two Phases of LLM Inference

- **Prefill**: process the prompt in parallel — **compute-bound** (matmul-heavy).
- **Decode**: generate tokens autoregressively, one at a time — **memory-bound** (read full KV cache + weights per token).
- This dichotomy is *why* PagedAttention helps (memory efficiency in decode), *why* continuous batching helps (fill decode-time GPU idleness), *why* speculative decoding helps (parallelize decode), and *why* prefill-decode disaggregation is now SOTA (different optimal hardware for each phase).

## 4.2 KV Cache & PagedAttention (vLLM)

The KV cache stores K/V for every past token at every layer × every head. For Llama-13B at 2048 context, KV cache is ~1.6 GB/sequence. Naive serving pre-allocates max-context per sequence → 60–80% memory wasted (internal + external frag + over-reservation).

**PagedAttention** (Kwon et al., SOSP 2023). Per the vLLM docs: *"The core idea of PagedAttention is to partition the KV cache of each request into KV Blocks. Each block contains the attention keys and values for a fixed number of tokens. The PagedAttention algorithm allows these blocks to be stored in non-contiguous physical memory."* Block size typically 16 tokens. Same block → same hash → automatic prefix sharing across requests (system-prompt reuse).
Per a recent performance study (Kolluru, arXiv 2511.17593): *"vLLM achieves up to 24× higher throughput than TGI under high-concurrency workloads through its novel PagedAttention mechanism."*
Eviction in vLLM: LRU on blocks with refcount=0; ties broken by "block at end of longest prefix."

**Grill-down (vLLM/PagedAttention)**
- L1: "Why is vLLM faster than HuggingFace?"
- L2: "Explain PagedAttention."
- L3: "Block size 16 — why not 1, why not 1024?" *(Too small: per-block metadata + kernel-launch overhead. Too large: frag returns; less sharing granularity.)*
- L4: "How does prefix caching work in vLLM, and what's the eviction policy?"
- L5: "Compare PagedAttention vs vAttention (CUDA VMM)." *(vAttention keeps virtual contiguity using CUDA VMM APIs — works with unmodified attention kernels at the cost of OS-roundtrip allocation latency, mitigated by overlapping with compute, opportunistic prefetch, and 64KB pages.)*

## 4.3 FlashAttention

FlashAttention (Dao et al., NeurIPS 2022) is an IO-aware exact-attention algorithm: tile Q/K/V into SMEM, online-softmax trick to avoid materializing the N×N matrix in HBM. Memory O(N²) → O(N); wall-clock 2–4× over vanilla.
**FlashAttention-2** (2023): parallelize over sequence length; inner loop over K/V blocks for better occupancy.
**FlashAttention-3** (Shah, Dao et al., 2024): per the Tri Dao blog, *"three main techniques to speed up attention on Hopper GPUs: exploiting asynchrony of the Tensor Cores and TMA to (1) overlap overall computation and data movement via warp-specialization and (2) interleave block-wise matmul and softmax operations, and (3) incoherent processing that leverages hardware support for FP8 low-precision."* Result: 75% of H100 peak (740 TFLOPS in FP16); 1.2 PFLOPS in FP8.

## 4.4 Continuous Batching

Static batching: wait for B requests, run together, all finish when slowest finishes. GPU wasted.
**Continuous batching** (Orca → vLLM): at each decode step, swap a new prompt into a slot that just emitted EOS. 5–20× throughput improvement.

## 4.5 Quantization

- **PTQ** vs **QAT**.
- **FP16/BF16**: half precision, default for training.
- **FP8** (E4M3/E5M2): Hopper+; training-grade with care.
- **INT8** (LLM.int8(), SmoothQuant): 8-bit weight + 8-bit activation; near-free with calibration.
- **INT4**: **GPTQ** (Frantar et al. — layer-wise reconstruction with approx Hessians); **AWQ** (Lin et al. — protect outlier-activated channels via per-channel scaling pre-quant). ~3.5× smaller, often <1% quality drop.
- **NF4** (QLoRA — Dettmers et al.): 4-bit NormalFloat for normally-distributed weights; basis of consumer-GPU fine-tuning.

**Grill-down.** "AWQ vs GPTQ?" *(GPTQ: layer-wise reconstruction loss + approx Hessians; mature tooling. AWQ: identifies salient outlier channels, protects them via scaling; tends to preserve instruction-following slightly better.)*

## 4.6 Speculative Decoding

Small draft model proposes K tokens; large target verifies them in one forward pass; accepts run, mismatched ones fall back. Per Google Research, the output distribution is identical to greedy/sampling from the target — i.e., quality-preserving. Speedup depends on **acceptance rate α**: a 70B target with a tuned 7B draft commonly yields 2–3×; mismatched drafts can be slower than baseline.
Variants: **Medusa** (decoding heads on target itself), EAGLE, lookahead decoding, prompt-lookup decoding (no draft, search the prompt).

## 4.7 Prefill-Decode Disaggregation

Disaggregate prefill (compute-bound, batch-friendly) onto one GPU pool; decode (memory-bandwidth-bound) on another; transfer KV cache between them (typically over RDMA). DistServe, SARATHI, TensorRT-LLM use variants. Allows independent scaling + mixed hardware (e.g., older H100 for decode, B200 for prefill).

## 4.8 LLM Serving Frameworks

| Framework | Strength | Pick when |
|---|---|---|
| **vLLM** | PagedAttention, continuous batching, broad model support | Default OSS serving |
| **SGLang** | RadixAttention (better prefix share), structured-gen DSL | Complex multi-turn agents, structured outputs |
| **TensorRT-LLM** | NVIDIA-optimized kernels, best raw throughput | Latency-critical, OK to compile |
| **TGI** (HF) | Mature, OpenAI-compatible API | Fast HF integration |
| **Triton + vLLM backend** | Multi-model serving + dynamic batching | Production multi-tenant |
| **llama.cpp** | GGUF; CPU/Metal/CUDA; quantized | Edge / on-device |
| **MLC LLM** | Compile to web, mobile | Browser/mobile |

## 4.9 Fine-Tuning Landscape

### Full vs PEFT
- **Full fine-tune**: best quality, expensive, catastrophic-forgetting risk.
- **LoRA** (Hu et al., ICLR 2022): freeze base; train low-rank `ΔW = BA` where `B ∈ R^{d×r}, A ∈ R^{r×d}, r << d`. Per the paper: *"LoRA can reduce the number of trainable parameters by a factor of 10,000 and the GPU memory requirement by a factor of 3"* (vs GPT-3 175B fine-tuned with Adam). Standard `r = 8–64`.
- **QLoRA** (Dettmers et al., NeurIPS 2023): 4-bit NF4 base + LoRA adapters in bf16; double quantization (~0.4 bits/param saving); paged optimizers. Enables 65B fine-tune on a single 48GB GPU.
- **DoRA, VeRA, PiSSA**: recent variants.

### Alignment
- **SFT**: instruction-following on (prompt, ideal_response) pairs.
- **RLHF / PPO** (InstructGPT, Ouyang et al. 2022): reward model on preference pairs → policy optimization with PPO + KL to reference. Unstable; 4 models in memory (policy, ref, reward, value).
- **DPO** (Rafailov et al., NeurIPS 2023, *"Your Language Model is Secretly a Reward Model"*): closed-form derivation that PPO + KL-regularized reward optimization reduces to a classification loss on preference pairs. No reward model, no RL loop. Default for most preference tuning in 2025–26.
- **GRPO** (DeepSeekMath / R1): sample G responses per prompt, normalize rewards within the group, drop the value model. The basis of DeepSeek-R1 reasoning training.
- **Constitutional AI / RLAIF** (Bai et al., Anthropic 2022): AI-judged preferences against a constitution.
- **KTO**: thumbs up/down without paired preferences.

### Decision framework — RAG vs Fine-tune vs Prompt
- Knowledge changes often / must be cited / auditable → **RAG**.
- New style / format / domain language → **fine-tune (LoRA)**.
- Solvable with clear instruction + few-shot + maybe one tool → **prompt + tool**.
- Most production systems combine all three.

## 4.10 RAG Architecture — Production Patterns

### Pipeline
1. **Ingest**: parse PDFs/HTML/MD (Unstructured.io, LlamaParse, Reducto) preserving structure.
2. **Chunk**: fixed-size + overlap (baseline 512 tokens, 50 overlap); recursive character splitter (LangChain); semantic chunking; **hierarchical** parent-child (retrieve small chunks, pass parent context to LLM) — current production default.
3. **Embed**: text-embedding-3-large (OpenAI), Cohere embed v3, BGE-large, voyage-3, GTE-large.
4. **Store**: **pgvector** (cheap, fits your Postgres-y stack), Pinecone (managed), Weaviate (hybrid first-class), Qdrant (Rust, fast), Milvus (scale), LanceDB (embedded), Vespa (enterprise).
5. **Retrieve**: **hybrid (BM25 + dense)** merged with Reciprocal Rank Fusion; metadata filter for tenant isolation; multi-query / query rewriting; **HyDE** (generate a hypothetical answer, embed *that*).
6. **Rerank**: cross-encoder (Cohere Rerank, BGE reranker, Jina) — slower, more precise. Retrieve top-100 dense → rerank to top-10.
7. **Generate**: stuff retrieved chunks; cite sources; structured output downstream.
8. **Cache**: semantic cache (Redis) on (query embed sim > 0.95 → reuse).

### Advanced patterns
- **HyDE** (Hypothetical Document Embeddings)
- **RAPTOR**: recursive abstractive summarization — tree of summaries; retrieve at the right level.
- **GraphRAG** (Microsoft): build a knowledge graph; retrieve a subgraph for global questions.
- **Self-RAG / CRAG**: model decides when to retrieve and when to retry.
- **Agentic RAG**: agent loops retrieve → eval → refine.

### RBAC for RAG (critical for enterprise FDE)
Don't put a user's allowed docs into one shared index. Options: (a) per-tenant index (high isolation, high ops cost); (b) shared index with hard metadata filter at query time *and* at storage layer (most common); (c) row-level security at the vector DB. ACL filter must run *before* ANN, not after.

**Grill-down (RAG)**
- L1: "What is RAG?"
- L2: "Why hybrid vs pure vector?" *(Vector misses exact tokens — error codes, product IDs, names. BM25 catches them. Merge with RRF.)*
- L3: "Walk me through your reranking choice. Why pay 200ms for a cross-encoder?"
- L4: "Customer reports 8% hallucination. Diagnose + fix." *(Add citations + faithfulness eval; check chunk recall@K — are the right chunks even being retrieved?; query rewriting; reranker; hierarchical chunks; prompt to refuse when context is insufficient; eval with RAGAS.)*
- L5: "Design RAG for a multi-tenant SaaS where some tenants are HIPAA, some GDPR, some airgapped. Embedding model choices, infra choices."

## 4.11 LLM Evaluation

- **Reference-based**: BLEU, ROUGE (n-gram overlap — weak); BERTScore, MoverScore (embedding-based — better); exact-match for extractive.
- **LLM-as-judge**: GPT-4/Claude with a rubric. Cheap, scalable. Caveats — position bias, length bias, self-preference; mitigate with pairwise + position swap + multiple judges.
- **RAGAS**: faithfulness, answer relevance, context precision, context recall, context relevance. Purpose-built for RAG.
- **TruLens / DeepEval / Phoenix / Langfuse / LangSmith / Braintrust / HoneyHive**: observability + eval frameworks.
- **Golden datasets**: hand-curated representative inputs + expected outputs. Run on every change. This is what FDEs build with customers in the first week of any engagement.
- **Online eval**: production traffic sampled + judged, closes the loop with offline goldens.
- **Eval harnesses**: lm-eval-harness, HELM, MMLU/GPQA/BIG-Bench.

**For agents, evaluate trajectory** (did it call the right tools in the right order?), not just final answer.

## 4.12 LLM Observability

- **Spans + traces** per agent invocation: prompt, completion, tool calls, latency, tokens, cost.
- Tools: **Langfuse** (OSS, self-hostable — fits your stack), **LangSmith** (LangChain-native), **Helicone** (OpenAI proxy), **Phoenix** (Arize), **Braintrust** (eval-first), **HoneyHive**.
- OpenTelemetry GenAI semantic conventions are stabilizing; use them to keep tooling pluggable.

## 4.13 Agents & MCP

### Agent patterns
- **ReAct** (Yao et al., ICLR 2023): interleaved Reason + Act + Observe.
- **Plan-and-Execute**: planner → executor.
- **Reflexion / Self-Refine**: critique own output, retry.
- **Tree of Thoughts**: branching with backtracking.
- **Multi-agent**: AutoGen (Microsoft), CrewAI, LangGraph, OpenAI Swarm — roles: planner, coder, reviewer, executor.

### Tool use / function calling
OpenAI / Anthropic function-calling APIs; JSON-schema tools; parallel tool calls. Anthropic's Claude Agent SDK has built-in primitives. **Vercel AI SDK has first-class tool calling — leverage your existing stack.**

### MCP (Model Context Protocol)
Anthropic-introduced (Nov 2024) open protocol over JSON-RPC 2.0. Three primitives: **tools** (model-controlled), **resources** (app-controlled), **prompts** (user-controlled). Transports: stdio (local), Streamable HTTP / SSE (remote). Adopted by OpenAI (March 2025) and Google DeepMind. SDKs in Python, TS, C#, Java.

**Anthropic explicitly lists "MCP servers, sub-agents, agent skills" as FDE deliverables** in its JDs — this is the single most directly relevant standard for an Anthropic FDE candidate.

Newer pattern: **MCP Code Execution** (Anthropic engineering, 2025). Instead of loading all tool definitions into the prompt (Anthropic's example: 150K tokens across many MCP servers), the agent discovers tools by exploring a `./servers/` filesystem and reading only the tool files it needs. Per Anthropic's post: *"This reduces the token usage from 150,000 tokens to 2,000 tokens — a time and cost saving of 98.7%."* Cloudflare calls this "Code Mode."

**Grill-down (MCP)**
- L1: "What's MCP?"
- L2: "Difference between an MCP tool, resource, and prompt?"
- L3: "Walk through the wire protocol of an MCP server advertising capability to a client."
- L4: "MCP security risks + mitigations." *(Prompt injection via tool descriptions; lookalike tools; over-privileged tokens. Mitigate: OAuth 2.1 with PRM, least-privilege scopes, signed tool manifests, log all tool invocations.)*
- L5: "Build an MCP server for a customer's Salesforce + internal SQL warehouse with row-level RBAC enforced via OAuth token passthrough."

## 4.14 Safety & Guardrails

- **I/O filtering**: NeMo Guardrails (programmable rails), Guardrails AI (validators), Llama Guard, Prompt Shield (Azure), Bedrock Guardrails.
- **Prompt injection defenses**: structured prompts that delimit system / user / tool; instruction-hierarchy training; spotlight prompts; never let untrusted text into a system-prompt slot.
- **Jailbreak detection**: classifiers on known jailbreaks; perplexity-based detection for gibberish.
- **Constitutional AI**: model self-critique against principles.
- **Red teaming**: structured adversarial testing — Anthropic + OpenAI both publish red-team playbooks.

## 4.15 Cost & Multi-Model Orchestration

- **Token economics**: input ~5–10× cheaper than output; cache it. Batch APIs (OpenAI, Anthropic) discount ~50% with 24h SLA.
- **Caching**: prompt caching (Anthropic API gives 90% input-token discount on cache hit); semantic cache.
- **Model routing**: cheap models (Haiku, gpt-4o-mini) for easy queries, top-tier for hard. RouteLLM, NotDiamond.
- **Fallbacks**: on rate limit / latency, fall back to alternate provider. LiteLLM, OpenRouter, Portkey.

---

# PHASE 5 — FDE-SPECIFIC EMPHASIS (Weeks 11–16)

## 5.1 Customer Deployment Patterns

| Pattern | When | Watch out for |
|---|---|---|
| **SaaS (vendor cloud)** | Default, fastest | Data residency, multi-tenant noise |
| **BYOC / VPC peering** | Mid-size enterprise | IAM cross-account, PrivateLink, egress cost |
| **Single-tenant dedicated** | Compliance-sensitive | Cost; ops burden |
| **In-customer-VPC deployment** | Regulated industries | Their cloud, their IAM, their security review |
| **On-prem** | Defense, finance core | Air-gapped patches, no telemetry |
| **Air-gapped** | Classified, FedRAMP High | Offline model distribution, manual eval roundtrip |

For Anthropic Federal Civilian FDE (open listing as of May 2026), expect questions on FedRAMP High, IL-5/IL-6, GovCloud regions, Bedrock Government, model export controls.

## 5.2 Production Debugging — The Core FDE Skill

Senior FDEs are judged on speed-to-root-cause in unfamiliar systems. Drill these patterns:

- **Four-axis hypothesis** for every incident: model change | data change | infra change | client change. Get one fact about each axis fast.
- **Reading traces**: from request-ID at the edge → gateway → services → model call, with KV cache stats and token counts.
- **Profiling**: pyspy, py-spy live, Nsight Systems (`nsys`) for GPU, `gpustat`, DCGM. Triton Performance Analyzer for serving.
- **Common LLM prod failures**:
  - **p99 spike** → KV cache pressure / batch wait time / cold-start of a new replica.
  - **Hallucination rate up** → check retriever recall first (corpus drift? embedding model updated? index rebuild?).
  - **Cost spike** → tool-calling loop without recursion limit; runaway agent memory.
  - **Refusal rate up** → safety filter change; vendor-side model update.
  - **Streaming hangs** → corporate proxy buffering; check SSE keepalive.

## 5.3 The "Decomposition" / Ambiguous-Client-Scenario Round

This is the decisive round at Palantir, OpenAI, and Anthropic. The pattern:
- Fuzzy real-world problem ("a logistics firm wants an AI agent to reroute shipments"; "a bank wants structured data extracted from 30 years of unstructured loan files").
- 30–60 minutes. No single correct answer.
- **You're graded on**: how you scope, what clarifying questions you ask, how you structure tradeoffs, how you sequence (POC → pilot → production), how you handle curveballs ("the data is dirtier than they said"; "their security team won't allow cloud egress"), and how you'd measure success.

**The 6-step decomposition framework — memorize**:
1. **Restate** what you heard.
2. **Clarify** scope, success metric, constraints (data, compliance, timeline, $).
3. **Decompose** into 3–5 sub-problems.
4. **Sketch** architecture; list 3 risks per component.
5. **Sequence** milestones: week-1 POC → month-1 pilot → quarter-1 prod.
6. **Identify kill criteria** — what would make you walk away.

**Practice prompts (drill weekly)**:
- "A pharma company wants Claude to read 10,000 clinical trial PDFs and produce regulatory submission summaries. Where do you start?"
- "An insurance firm wants an agent to handle claim triage end-to-end. Build the eval suite first."
- "A bank wants gpt-4o behind their own VPC for KYC with full PII redaction and zero internet egress."
- "Customer says their RAG 'isn't working.' What are the first 5 questions you ask?"

## 5.4 Customer Success / Pre-sales

- **Discovery**: lead with "tell me about the workflow today," not "we have a model that…". Map as-is, then to-be.
- **Stakeholder map**: technical buyer (CTO/Head of Eng), economic buyer (CFO/VP business), end-user (workflow owner), security/compliance gate-keeper.
- **Demo discipline**: never demo on stale data; never demo something untested in the customer's shape of environment; always have a fallback for streaming.
- **Land and expand**: ship one workflow, measure impact, expand.
- **Working backward from a press release** (Amazon-style PR/FAQ) is a useful framing for complex deployments.

---

# PHASE 6 — COMPANY-BY-COMPANY EMPHASIS MATRIX

## 6.1 AI Labs

### Anthropic — Forward Deployed Engineer / Applied AI
- **Required signal**: 3–4+ years technical customer-facing or strong founder background (*"Former technical founders are also encouraged to apply"* — verbatim from Anthropic's JD; you qualify directly); Python + ideally TS/Java; production LLM experience including prompt engineering, agent dev, evals, deployment at scale; high agency under ambiguity.
- **Process** (per interviewing.io + customcareer.miami.edu, May 2026): recruiter screen → take-home (commonly a ~90-min CodeSignal multi-tier problem — "implement a bank with multiple transaction types" is the widely circulated example) → hiring-manager call → onsite Loop 1 (system design + coding + culture fit; **Loop 2 cancelled if Loop 1 fails**) → Loop 2 (project deep-dive + experiences/goals). Coding done in **Google Colab / Replit / shared Python** — practical, not LeetCode. LLM round: prompt engineering, token/rate-limit handling, multi-step reasoning systems using LLM APIs.
- **Emphasis**: MCP servers, sub-agents, evals; Responsible Scaling Policy familiarity; ethical-pushback stories. The culture/values round is where most candidates fail (interviewers go 3–4 follow-ups deep).
- **Prep**: build a real MCP server end-to-end; build a Langfuse-instrumented + RAGAS-evaluated agent; read the Anthropic engineering blog ("Building effective agents," "Code execution with MCP"); read the MCP spec end-to-end.

### OpenAI — Forward Deployed Engineer
- **Process** (per Gaijineer firsthand 2025): recruiter (heavy on "why FDE, not just OpenAI") → ~5-hour take-home using OpenAI APIs (often RAG-style) → 60-min walkthrough → tech follow-up (chunking, retrieval quality, API rate-limiting, retry/backoff, eval design, debugging high-latency LLM pipelines) → customer-facing role play. Per OpenAI's official `interview-guide` page: final loop is *"4–6 hours of final interviews with 4–6 people over 1–2 days."*
- System design: payment-gateway-style (auth, queueing, **retries, idempotency**, monitoring); GPU scheduling system; GPU credit calculator.
- **Salary band**: $160K–$280K mid-level SF FDE per Marktechpost (May 2026).
- **Emphasis**: pragmatic coding (build an SQL engine, time-based KV store, resumable iterator); ownership; mission alignment; *"eval-driven feedback that changes product and model roadmaps."*

### Google DeepMind / Google Cloud FDE
- Standard Google SWE loop + customer-scenario rounds.
- Cloud FDE listings (Thomas Kurian, May 12 2026 LinkedIn): bands $127K–$265K (FDE II–IV), 59 roles week-one across US/India/Brazil/Australia/Mexico/Singapore/South Korea/Canada. Emphasis on Vertex AI, Gemini, Agentspace, customer co-build of agentic solutions.

## 6.2 FAANG / SaaS Giants

| Company | Most relevant role | Distinctive emphasis |
|---|---|---|
| **Meta** | ML Platform Eng / Production Eng | Hydra / FBLearner / Velox; C++ depth; B-user scale; impact-framed behavioral |
| **Amazon** | SRE / SA (AWS Pro Services) | Leadership Principles (rehearse STAR for all 16); operational excellence; AWS deep |
| **Apple** | ML Platform / Siri / Foundation Models | Privacy + on-device; mlx; quiet, deep technical loops |
| **Netflix** | ML Platform / SRE | Context-not-control culture; Spinnaker; chaos eng; Metaflow |
| **Google** | SRE / ML Engineer / Cloud FDE | SLO/error-budget fluency; system design; algorithmic still strong |
| **Microsoft** | Azure ML SA / OpenAI partner roles | Azure-stack literacy; enterprise compliance |

## 6.3 Data Engineering Giants

### Databricks — Solutions Architect / Resident SA
- **Process**: recruiter → tech screen → 3–5 onsite rounds: pair-programming/SQL + system design + customer role-play + behavioral. Per Dataford + DataCamp: Spark narrow vs wide; late-arriving data in streaming; Delta Lake ACID + time travel; cluster sizing + cost; multi-cluster warehouses; MLflow; competitive positioning vs Snowflake/Redshift/BigQuery. Customer role-play is decisive (discovery, value-prop, objection handling).
- **For you**: pitch the GEO Code / Shopify pipeline as "data + ML in production"; learn Spark/Delta/Unity Catalog; practice the "convince a skeptical client" dialogue.

### Snowflake — Solutions Architect
- **Process**: 3–4 rounds — Snowflake architecture (storage/compute separation, virtual warehouses, micro-partitions, Snowpipe), case studies, customer interaction simulation. Heavy: on-prem migration, security/governance, performance, cost.
- **2026 wrinkle**: Cortex (in-warehouse LLM), Snowpark, Iceberg open tables.

## 6.4 Consulting Giants

### McKinsey QuantumBlack — AI/ML Engineer
- **Process** (per hackingthecaseinterview + Glassdoor): QuantHub multi-choice (~12 Qs: programming, stats, modeling) or HackerRank → **Technical Experience Interview (TEI)** — 30-min project walkthrough + dive (often 2 TEI rounds) → ML/business case → McKinsey **PEI** (Personal Experience Interview, behavioral with leadership/impact).
- **Emphasis**: model interpretability for client storytelling; pair-programming clean Python; GCP/AWS/Databricks; Airflow; Docker/K8s. Case style is model-y (improve a prediction, explain the mechanism), not profitability-y.

### BCG GAMMA / BCG X — AI Engineer / Data Scientist
- **Process** (per Medium "sako" firsthand + hackingthecaseinterview): **CodeSignal coding (90 min, Python only**, scikit-learn/pandas/NumPy allowed, ~10 Qs across probability/stats/ML; ~60–70% accuracy passes; AI tools forbidden) → live coding (15 min, 2 data-manipulation tasks) + **technical case (30 min** — real client problem + propose ML approach) → ML system design (scalable pipelines, AWS/GCP/Azure, K8s, Docker, Terraform) → business case interviews → partner round. ~10–15% past first round; ~30% finish the loop with offer.

### Deloitte AI / Accenture AI Consultant
- HireVue/phone → tech screen → manager/case → partner/HR. Lower depth than FAANG/labs; **heavy on MLOps + cloud deployment** (Azure/AWS), CI/CD for ML, model monitoring, versioning, retraining, RAG/inference engineering basics. Accenture leans Azure AI + GenAI.
- **For you**: easiest entry by technical bar; slowest upside vs the labs.

## 6.5 Finance Giants

### Goldman Sachs ML Engineer
- HackerRank (3 problems, 120 min) → recruiter → CoderPad live phone screen → Super Day, 4 onsite rounds (coding, system design, ML round, behavioral). Coding includes LC-hard. ML round: deep-dive on past projects + concepts. System design example: web scraping engine, LinkedIn-style. Behavioral: Goldman Business Principles.

### JPMorgan ML Engineer
- HackerRank/CodeSignal → recruiter → 3–5 round onsite. Heavy MLOps: feature stores, online serving, latency budgets, monitoring, rollback; fraud/risk-scoring framing. Real ML system-design Q: *"Design a machine learning system that extracts from reddit/stocks daily and generates insight."* VP-led deep dives on PCA, eigen decomposition, p-values, chi-square. Final can be a 2-hour 4-VP panel.

### Citadel / Citadel Securities — Quant Dev / Engineer
- **Per citadelsecurities.com**: 45–60-min remote CoderPad. *"Our core programming languages are Python and C++, but we welcome any languages."* Tests DSA + problem-solving.
- **Quant Research** (per Matt Mitro, global head of grad recruitment, eFinancialCareers Jan 2023): 4 technical + 3 senior-management rounds with live coding embedded.
- ML-adjacent quant: high-level principles dive on past work + real-time coding. Low-latency system design appears for senior SWE/quant-dev (websockets vs REST, DB design, time complexity).

### Jane Street — Software Engineer
- **Official** (janestreet.com): *"We don't ask software engineers to do mental math, or math olympiad questions, or to contemplate logic puzzles. SWE interviews are about programming, plain and simple… we don't award bonus points for using a functional language like OCaml."*
- Process: OA → 45-min Zoom phone screens → NYC/London Super Day (collaborative programming). ML Engineer track: trading-flavored coding (rolling z-score, EMAs, resampling). Puzzles/probability live in the trader/research track, not SWE.

### Two Sigma
- HackerRank OA (75–180 min, 2 problems, LC medium–hard) → phone screen → onsite (multiple 60-min rounds + behavioral). Heavy emphasis on engineering perspective (clean, scalable, bug-free, optimal complexity). Track choices include Platform / Modeling / Data / Reliability / Trading / Enterprise / Infrastructure / Security Engineering.

---

# CANONICAL READING LIST (print + check off)

### Books — in order
1. **Site Reliability Engineering** (Beyer et al., Google) — free at sre.google.
2. **The Site Reliability Workbook** — operational SLO/error-budget practice.
3. **Designing Data-Intensive Applications** (Kleppmann) — single most important infra-adjacent book.
4. **Designing Machine Learning Systems** (Chip Huyen) — canonical MLOps.
5. **Machine Learning Engineering** (Andriy Burkov) — practical complement.
6. **Building Machine Learning Powered Applications** (Emmanuel Ameisen).
7. **Kubernetes Up and Running** (Hightower/Burns/Beda).
8. **The Linux Programming Interface** (Kerrisk) — reference.
9. **Systems Performance, 2nd ed.** (Brendan Gregg).
10. **Reliable Machine Learning** (Chen et al., O'Reilly).
11. **AI Engineering** (Chip Huyen, 2024) — the LLMOps companion.

### Papers — required reading
- **Attention Is All You Need** (Vaswani et al., 2017).
- **GPT-3 / Language Models are Few-Shot Learners** (Brown et al., 2020).
- **InstructGPT** (Ouyang et al., 2022) — the RLHF paper.
- **Constitutional AI** (Bai et al., Anthropic 2022) — RLAIF.
- **LoRA** (Hu et al., ICLR 2022, arXiv 2106.09685).
- **QLoRA** (Dettmers et al., NeurIPS 2023).
- **DPO** (Rafailov et al., NeurIPS 2023).
- **FlashAttention** (Dao et al., NeurIPS 2022); **FlashAttention-2** (Dao, 2023); **FlashAttention-3** (Shah, Dao et al., arXiv 2407.08608, 2024).
- **Efficient Memory Management for LLM Serving with PagedAttention** (Kwon et al., SOSP 2023) — the vLLM paper.
- **Orca: A Distributed Serving System for Transformer-Based Generative Models** (Yu et al., OSDI 2022) — continuous batching.
- **Fast Inference from Transformers via Speculative Decoding** (Leviathan, Kalman, Matias, ICML 2023).
- **ZeRO** (Rajbhandari et al., 2020); **ZeRO-Infinity** (SC 2021).
- **Megatron-LM** (Shoeybi et al., 2019) — tensor parallelism.
- **Hierarchical Navigable Small World** (Malkov & Yashunin, 2016).
- **REALM** (Guu et al., 2020) and **RAG** (Lewis et al., NeurIPS 2020).
- **Toolformer** (Schick et al., NeurIPS 2023).
- **ReAct** (Yao et al., ICLR 2023).
- **DeepSeekMath / GRPO** (Shao et al., 2024).
- **The Llama 3 Herd of Models** (Meta, 2024) — best single read on industrial-scale training.

### Blogs / engineering posts to subscribe to
- Anthropic engineering (anthropic.com/engineering)
- OpenAI engineering / research
- Google Research
- NVIDIA Developer blog (Triton, NCCL, TensorRT-LLM)
- Tri Dao (tridao.me)
- Chip Huyen (huyenchip.com)
- Lilian Weng (lilianweng.github.io)
- vLLM blog
- The Pragmatic Engineer (Gergely Orosz) — has covered FDE roles at OpenAI/Palantir
- Sundeep Teki (sundeepteki.org) — FDE career guides
- Latent Space (swyx + Alessio)

---

# 16-WEEK PLAN — Satvik-Calibrated

Assumes ~15 focused hours/week alongside founder work.

### Weeks 1–2: Foundation reset
- **Code daily**: 30 LeetCode-medium Python (hash maps, heaps, two pointers, intervals, BFS/DFS). Skip hard.
- **Read**: SRE book Ch. 1–6; DDIA Ch. 1–3.
- **Build**: containerize GEO Code, deploy to K8s (kind/minikube → EKS free-tier). Add Prometheus + Grafana. Define an SLO and an error-budget policy.
- **Watch**: Brendan Gregg's Systems Performance talks.

### Weeks 3–4: Linux + K8s + IaC
- **Read**: DDIA Ch. 5–9; *Kubernetes Up and Running*.
- **Build**: Terraform module standing up a GPU node group + S3 model store + IAM/IRSA + Secrets Manager.
- **Drill**: K8s grill-down chains (debug CrashLoopBackOff; write a Helm chart; install ArgoCD; ship GitOps).

### Weeks 5–6: MLOps fundamentals
- **Read**: Huyen *Designing ML Systems* cover-to-cover; Burkov.
- **Build**: end-to-end MLflow-tracked pipeline (any HF model) + register + serve via Triton + monitor with Evidently.
- **Drill**: feature-store concepts; point-in-time-correct SQL; drift metrics.

### Weeks 7–8: LLM internals & inference
- **Read**: vLLM paper; FlashAttention 1/2/3; Orca; speculative decoding paper; Huyen *AI Engineering*.
- **Build**: vLLM serving a Llama-3-8B-Instruct on a rented H100 (Modal/RunPod/Lambda Labs); benchmark TTFT + TPOT + throughput; tune batch size + max-model-len; enable prefix caching; measure $/M tokens.
- **Drill**: KV cache math; AWQ vs GPTQ vs FP8 head-to-head on the same model.

### Weeks 9–10: Fine-tuning + RAG
- **Read**: LoRA, QLoRA, DPO papers.
- **Build**: LoRA-fine-tune a 7B for a domain task (sanitized EY/tax assistant or GEO Code data); LLM-as-judge harness; RAGAS-evaluated RAG over real corpus (Shopify docs or tax code) with hybrid search + reranking.
- **Drill**: HNSW grill-down; RAG failure-mode triage.

### Weeks 11–12: Agents + MCP + production patterns
- **Build**: an open-source MCP server exposing a real workflow tool (Shopify integration or sanitized tax-question tool); a Claude-driven agent that uses it; Langfuse traces; Phoenix/RAGAS evals; a hand-curated golden dataset.
- **Read**: Anthropic engineering posts ("Building effective agents," "Code execution with MCP"); MCP spec.
- **Practice**: the 6-step decomposition framework on 10 practice prompts.

### Weeks 13–14: Mock-loop sprint
- 2 mock decomposition rounds/week (record + review).
- 3 mock system designs/week (LLM serving; RAG at scale; agent platform; multi-tenant feature store; multi-region inference).
- Anthropic-style CodeSignal practice (multi-tier, 90-min budget).
- OpenAI-style 5-hour mini take-home: rebuild a RAG over a public corpus in <5 hours with full evals + docs.

### Weeks 15–16: Apply + iterate
- Apply broadly: Anthropic, OpenAI, Scale AI, Palantir, Databricks Resident SA, Google Cloud FDE; plus 2 finance + 2 consulting + 2 FAANG SRE as safety net.
- Tailor each: founder narrative for Anthropic; production-ML narrative for Databricks; ambiguity/agency narrative for Palantir; LLM-product depth for OpenAI.
- Have 5 STAR stories ready: (1) hardest production debugging; (2) customer escalation; (3) shipped under ambiguity; (4) ethical pushback; (5) a system you designed that scaled.

---

## Recommendations (Staged + Concrete)

**This week**:
1. Read the Google SRE workbook SLO/error-budget chapter end-to-end + the vLLM PagedAttention paper.
2. Spin up vLLM on a rented H100 (Modal/RunPod/Lambda, ~$2–3/hour) and benchmark a Llama-3-8B against bare HF Transformers. This is the single most credible signal you can ship in a week.
3. Write an MCP server for one tool you use in GEO Code. Open-source it. Link it in your Anthropic application — it directly matches their JD ("Deliver technical artifacts for customers like MCP servers").

**Next 4 weeks**:
4. Containerize + deploy + observe one of your existing workloads to production K8s with Prometheus + Grafana + a written SLO + an error-budget policy. This is your DevOps/SRE proof-of-life.
5. Pick one production LLM workflow you've shipped at EY (sanitized) and write a 1-page case study: business problem → architecture → eval framework → observed metrics → what you'd do differently. This becomes your TEI / project-deep-dive for *every* interview.

**Threshold to start applying** (Week 8 if you're sprinting; Week 12 if pacing):
- You can debug a CrashLoopBackOff cold without StackOverflow.
- You can explain PagedAttention, FlashAttention-3, and continuous batching to a friend in plain English, then to a kernel engineer in detail.
- You have one open-source MCP server + one end-to-end RAG-with-evals on GitHub.
- You can run the 6-step decomposition on a fresh prompt in 45 minutes without panic.

**Threshold to walk away from a process**:
- If a "consulting" loop has zero technical depth and you'd be a body-shop resource — pass; you'll regress.
- If the hiring manager can't articulate the production AI problem they want you to solve — pass; the FDE role only works when the problem is real.
- If the salary band offered in India is materially below market (<60% of comparable Bengaluru AI-lab tier) without equity to compensate — counter or walk; FDE leverage is highest in your first 2 years.

---

## Caveats

- **The FDE market in May 2026 is hot, but also faddy.** Hiring bars are inconsistent across vendors. Filter for: production code expected, MCP/agents/evals mentioned in the JD, real frontier-model partnership, real travel + customer engagements. Avoid "FDE" titles that are essentially solutions-engineering re-branded.
- **Salary ranges in this doc are publicly reported and US-centric.** Bengaluru bands will be a fraction (often 20–40% of US numbers locally; more if you land remote-from-India at a frontier lab). Per Hashnode's 2026 FDE guide, Palantir average TC is ~$238K with staff-level FDEs clearing $630K+, OpenAI and Anthropic stabilized at $350K–$550K mid-to-senior — treat as direction, not destination.
- **LLM internals are moving fast.** This document is accurate to May 2026 — vLLM v0.9, FlashAttention-3 SOTA on Hopper, MCP spec 2025-11-25, OpenAI's Deployment Company launch (May 11, 2026), Google Cloud's FDE hiring push (May 12, 2026). Revisit the inference-internals and MCP sections every 3 months; SOTA on SSMs/Mamba, MoE serving, B200 quirks, NVLink Switch System patterns will shift.
- **Some sources cited are aggregator sites (Glassdoor, Interview Query, datainterview, IGotAnOffer, fde.academy, Dataford).** Directionally reliable for process shape; specific question content is often paraphrased and dated. Cross-check against the official career pages where possible: Anthropic Greenhouse JDs, openai.com/interview-guide, janestreet.com/preparing-for-a-software-engineering-interview, citadelsecurities.com/careers/career-perspectives/our-engineering-interview-process — all primary.
- **Anthropic's FDE listings explicitly invite "former technical founders"** — verbatim. This is the cleanest entry signal for your background. Lead with the founder story, not the EY consulting story, when you write your Anthropic application.