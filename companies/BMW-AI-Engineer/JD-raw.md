# BMW Tech Works — Python/Prompt Engineer (LLM-Based AI Agents)

| | |
|---|---|
| **Company** | BMW Tech Works India (BTWI) — DECO unit |
| **Location** | Pune |
| **Posted / Closes** | 2026-03-30 / 2026-06-30 |
| **Experience required** | 6–10 years |
| **Working hours** | 5×9 (Mon–Fri, CET/CEST 08:00–17:00) |
| **Job code** | BTWI/-E-AA/1709884 |
| **Domain** | Enterprise process automation across quality, logistics, production, business |

## Role Summary
Build production agentic systems (single + multi-agent) on top of LLMs to automate enterprise processes inside BMW. Heavy emphasis on classic agent patterns (ReAct, Planner/Executor, Reflection), RAG, tool-wrapping, evaluation, and shipping through Docker/K8s/GitHub CI/CD. Senior level — architect agent systems, not just call APIs. Two halves: AI (agent design, prompting, RAG, eval, safety) + Engineering (Python at scale, FastAPI, container orchestration, observability).

## Topic Areas
1. Agent core concepts (ReAct, CoT, Reflection, ToT, Planner/Executor, tool use, memory, guardrails) — 40% interview weight
2. Python engineering (asyncio, OOP, FastAPI, Pydantic v2, testing, packaging)
3. RAG (ingest → chunk → embed → index → retrieve → rerank → generate → cite; FAISS, Chroma, Milvus, Pinecone; Ragas eval)
4. Multi-agent systems (MCP, A2A, LangGraph, CrewAI, AutoGen, DSPy)
5. MLOps & deployment (Docker, K8s, GitHub/Jenkins CI/CD, Prometheus, Grafana, ELK, MLflow)
6. Evaluation (DeepEval, Ragas, TruLens, Promptfoo, Guardrails AI, LLM-as-judge)
7. Prompt engineering (zero/few-shot, CoT variants, dynamic few-shot, self-ask, compression)
8. Data integration (SQL agents, text-to-SQL, Kafka, ERP/SAP, XML/JSON)
9. Safety & security (prompt injection, jailbreaks, RBAC for agents, GDPR, EU AI Act, audit logging)
10. Frameworks comparison (LangChain, LlamaIndex, Haystack, CrewAI, AutoGen, DSPy)

## Interview Themes (from JD)
- System design: "Design an agent that automates warranty-claim triage from dealer emails"
- System design: "Design RAG over 10 years of internal repair manuals (PDFs with diagrams)"
- System design: "Multi-agent system — planner routes logistics, executor checks supplier ETAs"
- Mechanism: "Walk me through ReAct vs CoT. When does ReAct fail?"
- Mechanism: "Compare HNSW vs IVF-PQ. When to pick each?"
- Trade-off: "LangChain vs custom agent loop — defend your choice"
- Trade-off: "Fine-tune vs RAG vs few-shot — how do you decide?"
- Production: "Agent works in dev, fails in prod 20% of the time — debug it"
- Production: "Token spend doubled overnight — what do you check?"
- BMW-flavor: "Where would agents NOT be appropriate in manufacturing?"

## Stacks Mentioned
Languages: Python | Frameworks: FastAPI, LangChain, LlamaIndex, Haystack, CrewAI, AutoGen, DSPy, LangGraph
Infra: Docker, Kubernetes, GitHub Actions, Jenkins | Data: Kafka, SQL, XML/JSON, SAP/ERP
AI: OpenAI, Anthropic APIs, FAISS, Chroma, Milvus, Pinecone, MLflow
Eval: Ragas, DeepEval, TruLens, Promptfoo, Guardrails AI
Observability: Prometheus, Grafana, ELK, OpenTelemetry
Protocols: MCP (Anthropic), A2A (Google)
