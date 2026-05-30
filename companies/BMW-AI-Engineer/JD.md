# BMW Tech Works India — Python/Prompt Engineer (LLM-Based AI Agents)
*Extracted: 2026-05-30 | Job code: BTWI/-E-AA/1709884*

## Role Summary
Senior-level (6–10 YoE) role in BTWI's DECO unit. Build production agentic AI systems to automate enterprise processes across quality, logistics, production, and business. Equal weight on AI architecture (agent design, RAG, eval, safety) and engineering (Python, FastAPI, containers, CI/CD, observability).

## Seniority
**Senior** — expected to architect agent systems end-to-end, not just consume APIs.

## Technical Stack
| Category | Technologies |
|----------|-------------|
| Languages | Python (expert-level) |
| Agent Frameworks | LangChain, LangGraph, LlamaIndex, Haystack, CrewAI, AutoGen, DSPy |
| LLM APIs | OpenAI, Anthropic (Claude) |
| RAG / Vector | FAISS, Chroma, Milvus, Pinecone |
| Web / Services | FastAPI, async Python, Pydantic v2 |
| Data | Kafka, SQL, XML/JSON, SAP/ERP |
| Infra | Docker, Kubernetes, GitHub Actions, Jenkins |
| Observability | Prometheus, Grafana, ELK, OpenTelemetry |
| Eval | Ragas, DeepEval, TruLens, Promptfoo, Guardrails AI |
| MLOps | MLflow, model registries |
| Protocols | MCP (Anthropic), A2A (Google) |

## Core CS Requirements — Primer Mapping
| Requirement | Maps to Primer |
|-------------|---------------|
| LLMs, agents, RAG, prompt engineering | AI-ML & Gen AI |
| MLOps, Docker, K8s, CI/CD, observability | DevOps |
| Microservices, API design, message queues (Kafka) | System Design |
| SQL agents, vector DBs | DBMS |
| Async Python, concurrency | OS (process/thread) |
| FastAPI, HTTP/2, WebSockets | Computer Networks |

## Managerial / Leadership Requirements
- Senior technical contributor; architect-level ownership of agent systems
- Cross-functional collaboration with quality, logistics, production teams
- Expectation to drive decisions on framework choice, eval strategy, safety policy
- Not a people-manager role based on JD language

## Inferred Interview Format
*Researched from BMW/BTWI patterns on Glassdoor/Blind/LinkedIn:*

| Round | Type | Focus | Est. Duration |
|-------|------|-------|---------------|
| 1 | HR Screening | Background, motivation, logistics | 30 min |
| 2 | Technical — Python + Agents | asyncio, OOP, ReAct, tool use, CoT | 60 min |
| 3 | Technical — RAG + Eval | RAG pipeline design, Ragas, DeepEval, LLM-as-judge | 60 min |
| 4 | System Design | Full multi-agent system design at automotive scale | 60 min |
| 5 | Behavioral / Culture | STAR stories, cross-functional scenarios, failure examples | 45 min |
| 6 | Manager / Final | Architecture decisions, trade-offs, roadmap thinking | 45 min |

## Key Differentiator
Many candidates strong on *either* AI *or* engineering. The offer goes to someone strong on **both**. Focus equal prep time on each half.

## BMW-Specific Context
- BMW runs SAP heavily — ERP tool integration is realistic
- EU automotive = serious about GDPR, EU AI Act, audit logging
- Manufacturing context = agents must be deterministic and certifiable in some domains
- Internal documents include CAD/PDFs with diagrams — multi-modal RAG is plausible

## Sources
- JD pasted directly (2026-05-30)
- BMW BTWI Glassdoor reviews: interview typically 3–5 rounds, technical heavy
- LeetCode Discuss: BTWI tends to focus on system design + practical coding, not LeetCode-style DSA
