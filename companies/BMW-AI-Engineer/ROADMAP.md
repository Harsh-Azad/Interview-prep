# Prep Roadmap — BMW AI Engineer
*Generated: 2026-05-30 | Target: 6 weeks (~10 hrs/week) | Closes: 2026-06-30*

---

## Priority Tiers

### Tier 1 — Must Know (JD explicitly requires, ~40% of interview time)
- [ ] Agent Patterns — ReAct, CoT, Reflection, ToT, Planner/Executor → `AI-ML_Gen_AI_Advance_Primer.md §12.1`
- [ ] Tool Use & Function Calling (OpenAI vs Anthropic schemas, Pydantic structured output) → `§12.2`
- [ ] Agent Memory — short-term buffer, long-term vector store, summary memory → `§12.7`
- [ ] Multi-Agent Systems — MCP protocol, A2A protocol, LangGraph state machines → `§12.3, §12.5`
- [ ] RAG Pipeline — ingest → chunk → embed → index → hybrid retrieve → rerank → generate → cite → `§11.1–§11.12`
- [ ] Vector Stores — FAISS, Chroma, Milvus, Pinecone; ANN: HNSW vs IVF-PQ → `DBMS_Primer.md §19`
- [ ] Evaluation — Ragas metrics, DeepEval, TruLens, Promptfoo, LLM-as-judge → `AI-ML §10g, §12.9`
- [ ] Guardrails — input/output safety, schema validation, PII scrubbing, jailbreak defense → `§16.3, §16.4`
- [ ] Docker — multi-stage builds, distroless, security scanning → `Devops_Primer.md §2.1`
- [ ] Kubernetes — Deployments, Services, Ingress, HPA, StatefulSets → `Devops_Primer.md §2.2`
- [ ] CI/CD — GitHub Actions pipeline: lint → test → build → push → deploy → `Devops_Primer.md §2.3`
- [ ] Observability — Prometheus, Grafana, ELK, OpenTelemetry; LLM-specific traces → `Devops_Primer.md §2.6, §4.12`
- [ ] Prompt Engineering — zero/few-shot, dynamic few-shot, CoT variants, self-ask → `AI-ML §13.1–§13.7`
- [ ] SQL Agents — text-to-SQL, query validation, sandboxed execution → `DBMS_Primer.md §2`
- [ ] Kafka — producers/consumers, partitions, offset management, Python (aiokafka) → `DBMS_Primer.md §18`
- [ ] FastAPI — async routing, streaming SSE, background tasks, dependency injection → `System_Design_Primer.md §6`
- [ ] Async Python — asyncio, gather, Semaphore for rate limiting, Queue patterns → `OS §process/thread`
- [ ] Pydantic v2 — field validators, model validators, `instructor`, structured output → in-primer + beyond

### Tier 2 — Strong Signal (common at BMW / senior level)
- [ ] Fine-tuning & PEFT — LoRA, QLoRA, when fine-tune beats RAG → `AI-ML §fine-tuning`
- [ ] LLM Observability — LangSmith / Langfuse / Arize Phoenix span tracing → `Devops §4.12`
- [ ] DSPy — Signatures, Modules, Teleprompters; compare vs LangChain → `AI-ML §12.4`
- [ ] Microservices — circuit breakers, retry + jitter, bulkheads, service mesh → `System_Design §microservices`
- [ ] Prompt injection defense — instruction hierarchies, content sandboxing, output classifiers → `AI-ML §16.3`
- [ ] RBAC for agents — agent permissions ≤ user permissions; per-tenant isolation → `Devops §4.14`
- [ ] MLflow — experiment tracking, prompt versioning, model registry → `Devops §3.4`
- [ ] EU AI Act / GDPR — automated decision transparency, right to explanation, data minimization
- [ ] GraphRAG / KG-augmented — when relationships > chunks; Microsoft GraphRAG paper
- [ ] Multi-modal RAG — PDFs with diagrams, tables; automotive manuals use case

### Tier 3 — Good to Have
- [ ] Tree-of-Thoughts deep dive — branching search, when extra compute pays off
- [ ] GPU scheduling in K8s — node selectors, taints/tolerations, MIG
- [ ] CrewAI vs AutoGen vs LangGraph — comparison, when to pick each
- [ ] LlamaIndex vs LangChain — query engines, response synthesizers
- [ ] Synthetic eval data generation — LLM-generated test cases, bias risks
- [ ] SAP/ERP integration patterns — auth, transactional safety, idempotency

---

## Managerial / Behavioral Prep
- [ ] STAR: Led architectural decision that affected team direction
- [ ] STAR: Debugged a production failure under pressure
- [ ] STAR: Disagreed with a technical decision — how resolved
- [ ] STAR: Delivered a complex project with unclear requirements
- [ ] STAR: Failure story — what went wrong, what you learned
- [ ] BMW values: "Joy, Responsibility, Appreciation" — align stories to these

---

## Week-by-Week Plan

### Week 1 — Python Foundation (2026-05-30 → 2026-06-05)
| Day | Focus | Command |
|-----|-------|---------|
| 1–2 | asyncio, gather, Semaphore, Queue; common deadlock mistakes | `/deep-dive asyncio` |
| 3–4 | FastAPI — async routing, StreamingResponse, SSE, background tasks | `/deep-dive fastapi` |
| 4–5 | Pydantic v2 — validators, model_config, instructor for structured LLM output | `/deep-dive pydantic-v2` |
| Weekend | Build: FastAPI service that streams an LLM response with proper async + retries | `/sim devops easy` |

### Week 2 — Single-Agent Core (2026-06-06 → 2026-06-12)
| Day | Focus | Command |
|-----|-------|---------|
| 1–2 | ReAct pattern deep dive — thought/action/observation, failure modes | `/deep-dive react-agent` |
| 2–3 | CoT variants — zero-shot, few-shot, self-consistency; Tree-of-Thoughts | `/deep-dive chain-of-thought` |
| 3–4 | Tool use / function calling — OpenAI vs Anthropic schemas, Pydantic output | `/deep-dive tool-use` |
| 4–5 | Agent memory — buffer, vector store, summary; scratchpad token budgeting | `/deep-dive agent-memory` |
| Weekend | Build: ReAct agent from scratch (no framework), re-implement in LangGraph | `/sim ai hard` |

### Week 3 — RAG End-to-End (2026-06-13 → 2026-06-19)
| Day | Focus | Command |
|-----|-------|---------|
| 1–2 | Chunking strategies, embedding models, vector stores (FAISS/Chroma/Milvus) | `/deep-dive rag-pipeline` |
| 2–3 | ANN indexes — HNSW vs IVF-PQ; hybrid retrieval (BM25 + dense + reranker) | `/deep-dive vector-indexes` |
| 3–4 | Query rewriting, HyDE, context compression (LLMLingua) | `/deep-dive rag-advanced` |
| 4–5 | Ragas metrics — faithfulness, answer relevancy, context precision/recall | `/deep-dive ragas-eval` |
| Weekend | Build: RAG over the 6 primer files. Evaluate with Ragas. | `/sim ai hard` |

### Week 4 — Multi-Agent + MCP/A2A (2026-06-20 → 2026-06-25)
| Day | Focus | Command |
|-----|-------|---------|
| 1–2 | Multi-agent patterns — Supervisor, Planner, Executor, Critic roles | `/deep-dive multi-agent` |
| 2–3 | MCP protocol — client/server/host, transport layers, build a small MCP server | `/deep-dive mcp` |
| 3–4 | A2A protocol — Agent Card schema, Task lifecycle, push notifications | `/deep-dive a2a` |
| 4–5 | LangGraph — typed state, conditional edges, checkpointing, time-travel | `/deep-dive langgraph` |
| Weekend | Build: Small MCP server exposing tools + multi-agent crew using it | `/interview-round BMW-AI-Engineer system-design` |

### Week 5 — Eval + Observability + Safety (2026-06-26 → 2026-07-01)
| Day | Focus | Command |
|-----|-------|---------|
| 1–2 | DeepEval, TruLens, Promptfoo — when to use each; CI integration | `/deep-dive eval-frameworks` |
| 2–3 | Guardrails — NeMo Guardrails, Guardrails AI, Llama Guard; RBAC for agents | `/deep-dive guardrails` |
| 3–4 | Observability — Prometheus/Grafana, ELK, OpenTelemetry GenAI spans | `/deep-dive llm-observability` |
| 4–5 | EU AI Act, GDPR for AI systems, audit logging requirements | `/deep-dive eu-ai-act` |
| Weekend | Build: CI pipeline — Promptfoo + Ragas + Guardrails on every commit | `/interview-round BMW-AI-Engineer technical-2` |

### Week 6 — Frameworks + Mock Rounds (Final push before 2026-06-30)
| Day | Focus | Command |
|-----|-------|---------|
| 1–2 | LangChain/LangGraph deep, LlamaIndex vs LangChain comparison | `/deep-dive langchain-vs-llamaindex` |
| 2–3 | DSPy — Signatures, Modules, Teleprompters; compare paradigm to LangChain | `/deep-dive dspy` |
| 3 | SQL agents — text-to-SQL, Spider/BIRD benchmarks, execution-match eval | `/deep-dive sql-agents` |
| 4–5 | Full mock — all rounds back to back | `/interview-round BMW-AI-Engineer screening` |
| Weekend | Review weak spots from rounds, final `/revise` pass | `/revise` |

---

## Interview Round Schedule
| Round | Command | Priority |
|-------|---------|----------|
| Screening | `/interview-round BMW-AI-Engineer screening` | After Week 2 |
| Technical 1 (Python + Agents) | `/interview-round BMW-AI-Engineer technical-1` | After Week 2 |
| Technical 2 (RAG + Eval) | `/interview-round BMW-AI-Engineer technical-2` | After Week 3 |
| System Design | `/interview-round BMW-AI-Engineer system-design` | After Week 4 |
| Behavioral | `/interview-round BMW-AI-Engineer behavioral` | After Week 5 |
| Final / Manager | `/interview-round BMW-AI-Engineer managerial` | Week 6 |
