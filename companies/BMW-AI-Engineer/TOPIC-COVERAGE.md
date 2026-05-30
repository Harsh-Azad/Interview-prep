# Topic Coverage — BMW AI Engineer
*Synced from global TRACKER.md | Generated: 2026-05-30*
*Check boxes here AND in global `progress/TRACKER.md` when a topic is covered.*

---

## Tier 1 — Must Cover (18 topics from global TRACKER)

### From AI-ML & Gen AI Primer
- [ ] Large Language Models — LLMs ← `AI-ML_Gen_AI_Advance_Primer.md` | Global: AI-04
- [ ] RAG — Retrieval-Augmented Generation ← `AI-ML_Gen_AI_Advance_Primer.md` | Global: AI-06
- [ ] Prompt Engineering ← `AI-ML_Gen_AI_Advance_Primer.md` | Global: AI-08
- [ ] AI Evaluation & Benchmarks ← `AI-ML_Gen_AI_Advance_Primer.md` | Global: AI-09
- [ ] MLOps & Model Deployment ← `AI-ML_Gen_AI_Advance_Primer.md` | Global: AI-10
- [ ] Transformers & Attention Mechanism ← `AI-ML_Gen_AI_Advance_Primer.md` | Global: AI-03

### From DevOps Primer
- [ ] Docker & Containerization ← `Devops_Primer.md` | Global: DO-02
- [ ] Kubernetes Orchestration ← `Devops_Primer.md` | Global: DO-03
- [ ] CI/CD Pipelines ← `Devops_Primer.md` | Global: DO-01
- [ ] Monitoring & Observability ← `Devops_Primer.md` | Global: DO-05
- [ ] Logging & Distributed Tracing ← `Devops_Primer.md` | Global: DO-06

### From System Design Primer
- [ ] Message Queues & Event-Driven Architecture (Kafka) ← `System_Design_Primer.md` | Global: SD-07
- [ ] Microservices vs Monolith ← `System_Design_Primer.md` | Global: SD-08
- [ ] API Design (REST, GraphQL, gRPC) ← `System_Design_Primer.md` | Global: SD-09

### From DBMS Primer
- [ ] SQL vs NoSQL ← `DBMS_Primer.md` | Global: DB-02
- [ ] Vector Databases & Embeddings ← `DBMS_Primer.md` | Global: DB-10 (NewSQL/Modern DBs section)

### From OS Primer
- [ ] Process vs Thread (for async Python understanding) ← `Operating_System_Primer.md` | Global: OS-01

### From Computer Networks Primer
- [ ] HTTP / HTTPS / HTTP2 / HTTP3 ← `Computer_Networks_Primer.md` | Global: CN-03

---

## Tier 1 — BMW-Specific Topics (not in global TRACKER)
*These are role-specific additions tracked only here.*

- [ ] Agent Patterns — ReAct, Reflection, Self-Refine, Tree-of-Thoughts
- [ ] Planner / Controller / Executor architecture
- [ ] Tool Use & Function Calling — OpenAI vs Anthropic schemas, JSON mode vs Pydantic
- [ ] Agent Memory — buffer/window vs vector store vs summary; scratchpad token budgets
- [ ] Agent Budget Controls — max tool calls, max tokens/loop, wall-clock timeouts
- [ ] Idempotency for agent tool calls
- [ ] Multi-Agent System patterns — Supervisor, Planner, Executor, Critic roles
- [ ] MCP Protocol — client/server/host, stdio/SSE/HTTP transport, building an MCP server
- [ ] A2A Protocol — Agent Card schema, Task lifecycle, push notifications
- [ ] LangGraph — typed state schemas, conditional edges, checkpointing, time-travel debug
- [ ] FastAPI advanced — StreamingResponse, SSE, BackgroundTasks, WebSocket, OAuth2 middleware
- [ ] Pydantic v2 advanced — field validators, model validators, `instructor`, `outlines`
- [ ] Async Python deep — asyncio.Semaphore for LLM rate limiting, asyncio.Queue for agent pipelines
- [ ] Evaluation frameworks — Ragas (faithfulness/relevancy/precision/recall), DeepEval, TruLens, Promptfoo
- [ ] LLM-as-judge — position bias, length bias, calibration strategies
- [ ] Guardrails libraries — NeMo Guardrails, Guardrails AI, Llama Guard
- [ ] SQL Agents / Text-to-SQL — Spider/BIRD benchmarks, execution-match eval
- [ ] Kafka in Python — aiokafka (async), confluent-kafka-python; agent event bus patterns
- [ ] LLM Observability — LangSmith / Langfuse / Arize Phoenix; OpenTelemetry `gen_ai.*` spans
- [ ] Prompt injection defense — instruction hierarchies, content sandboxing, output classifiers
- [ ] RBAC for agents — agent permissions ≤ user permissions; per-tenant RAG isolation
- [ ] EU AI Act / GDPR for AI — high-risk classification, documentation obligations, right to explanation

---

## Tier 2 — Should Cover (10 topics from global TRACKER)

### From AI-ML & Gen AI Primer
- [ ] Fine-tuning & PEFT — LoRA, QLoRA; when fine-tune beats RAG ← AI-07
- [ ] Neural Networks & Deep Learning (background depth) ← AI-02

### From System Design Primer
- [ ] Caching Strategies (agent memory, KV cache, RAG caching) ← SD-03
- [ ] Distributed Systems Fundamentals ← SD-10

### From DBMS Primer
- [ ] Indexing & Query Optimization (SQL agent context) ← DB-03

### From Computer Networks Primer
- [ ] WebSockets & Long Polling (streaming agent responses) ← CN-08

### From DevOps Primer
- [ ] Infrastructure as Code (Terraform for agent infra) ← DO-04
- [ ] DevSecOps Basics (SAST, secrets management) ← DO-10
- [ ] Git Workflows & Branching Strategies ← DO-07

### Tier 2 — BMW-Specific
- [ ] DSPy — Signatures, Modules, Teleprompters; paradigm comparison vs LangChain
- [ ] GraphRAG / KG-augmented RAG — Microsoft GraphRAG paper; when relationships > chunks
- [ ] Multi-modal RAG — PDFs with diagrams/tables; automotive manuals context
- [ ] Synthetic eval data generation — LLM-as-test-case-generator; bias risks
- [ ] MLflow / prompt versioning — experiment tracking, prompt registry, canary releases for agents

---

## Tier 3 — Nice to Have

### From Global TRACKER
- [ ] GPU Scheduling in Kubernetes (node selectors, taints, MIG) ← DO-03 extension
- [ ] Service Mesh — Istio, mTLS, traffic policies ← DO-09
- [ ] OLTP vs OLAP (context for data integration) ← DB-09

### BMW-Specific
- [ ] SAP/ERP integration — auth patterns, transactional safety
- [ ] Tree-of-Thoughts deep dive — branching search, when extra compute is justified
- [ ] Canary releases for agents — % traffic shifting, eval-metric-based rollback
- [ ] CrewAI vs AutoGen vs LangGraph — side-by-side comparison
- [ ] LlamaIndex vs LangChain — query engines, response synthesizers

---

## Behavioral / STAR Bank
- [ ] STAR: Led architectural decision with major team impact
- [ ] STAR: Production failure — debug under pressure
- [ ] STAR: Disagreement on technical direction — how resolved
- [ ] STAR: Complex delivery with unclear requirements
- [ ] STAR: Failure story (mandatory — BMW will ask)
- [ ] STAR: Cross-functional collaboration story
- [ ] BMW context: "Why BMW / automotive AI specifically?"

---

## Coverage Progress
| Tier | Covered | Total | % |
|------|---------|-------|---|
| Tier 1 (global TRACKER) | 0 | 18 | 0% |
| Tier 1 (BMW-specific) | 0 | 22 | 0% |
| Tier 2 | 0 | 14 | 0% |
| Tier 3 | 0 | 8 | 0% |
| Behavioral | 0 | 7 | 0% |
| **Overall** | **0** | **69** | **0%** |
