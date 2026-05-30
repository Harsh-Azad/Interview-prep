# Mind Map — BMW AI Engineer
*Role-specific skill tree | Use `/deep-dive <concept>` to expand any node*
*✅ = covered | ⬜ = not yet*

---

## BMW AI Engineer Skill Tree

### Agent Core *(~40% interview weight)*
#### ReAct Pattern ⬜
- Thought → Action → Observation loop
- Failure modes: tool spam, infinite loops, reasoning drift
- When to use vs pure CoT

#### Chain-of-Thought ⬜
- Zero-shot CoT ("Let's think step by step")
- Few-shot CoT with curated exemplars
- Self-consistency: sample N times, majority vote
- Automatic CoT — auto-generate chain examples

#### Reflection / Reflexion / Self-Refine ⬜
- Agent re-reads trace → revises
- Cost vs accuracy trade-off
- When to add a Critic agent instead

#### Tree-of-Thoughts ⬜
- Branching search over reasoning steps
- BFS vs DFS traversal
- When extra compute justifies the cost

#### Planner / Controller / Executor ⬜
- Why not one model doing all three at scale
- What planner sees vs executor sees
- Failure modes: plan drift, executor hallucination

#### Tool Use & Function Calling ⬜
- OpenAI tool schema vs Anthropic tool_use blocks
- Pydantic + `instructor` for schema enforcement
- Parallel tool calls — handling race conditions
- Idempotency keys for mutation tools
- Error surface design: what to expose vs hide

#### Agent Memory ⬜
- Short-term: buffer window, sliding token window
- Long-term: vector store retrieval, summary memory
- Scratchpad: token budget vs reasoning quality
- External: files, databases, knowledge graphs

#### Budget Controls & Reliability ⬜
- Max tool calls per task
- Max tokens per loop iteration
- Wall-clock timeouts
- Self-healing: retry with reflection vs naive retry
- Determinism: temperature=0, seed, top_p=1

#### Guardrails ⬜
- Input filtering: intent classifiers, PII detection
- Output filtering: schema validation, refusal detection
- Jailbreak detection: DAN-style, role-play, encoding tricks
- RBAC: agent effective permissions = min(agent_caps, user_caps)

---

### Python Engineering
#### Async Python ⬜
- asyncio event loop mechanics
- `async def` / `await` / `asyncio.gather`
- `asyncio.Semaphore` — rate-limit LLM calls
- `asyncio.Queue` — producer-consumer agent pipelines
- When threads beat asyncio (CPU-bound); when asyncio wins (I/O-bound LLM calls)

#### FastAPI ⬜
- Dependency injection with `Depends`
- `StreamingResponse` + SSE for token streaming
- `BackgroundTasks` vs Celery vs Arq for long agent runs
- WebSocket for persistent agent sessions
- Lifespan events for startup/shutdown (model loading)
- OAuth2 / JWT middleware

#### Pydantic v2 ⬜
- Field validators, model validators
- `model_config` settings
- `instructor` library — structured LLM output
- `outlines` — constrained generation
- Why JSON mode alone ≠ schema safety (silent drift)

#### OOP & Design Patterns ⬜
- ABCs, Protocols (PEP 544)
- Strategy pattern → swappable LLM providers
- Factory pattern → dynamic tool instantiation
- Adapter pattern → wrapping external APIs as tools
- Dependency injection for testability

#### Testing ⬜
- pytest + pytest-asyncio
- Mocking LLMs: `respx` for httpx, VCR cassettes
- Property-based testing with `hypothesis`
- Evaluation tests as CI gates (Promptfoo)

---

### RAG Pipeline
#### Ingestion & Chunking ⬜
- Fixed-size, sentence, paragraph, semantic, recursive
- Structure-aware: Markdown headings, code blocks
- Overlap trade-offs
- Incremental indexing, change detection

#### Embedding Models ⬜
- text-embedding-3-large, Cohere, BGE, E5, Nomic
- Dimensionality vs retrieval quality
- Domain fine-tuning for automotive/enterprise

#### Vector Stores ⬜
- FAISS — in-process library; best for dev/small scale
- Chroma — embedded, dev-friendly
- Milvus — distributed, production scale
- Pinecone — fully managed, serverless option

#### ANN Indexes ⬜
- HNSW — most common, high recall, high memory
- IVF — coarse quantization, lower memory
- IVF-PQ — compressed vectors, lowest memory, some recall loss
- DiskANN — when index doesn't fit in RAM

#### Hybrid Retrieval ⬜
- BM25 (sparse) + dense vector search
- Reciprocal Rank Fusion (RRF)
- Cross-encoder reranker: latency cost vs quality gain

#### Advanced RAG ⬜
- HyDE — generate hypothetical document, embed that
- Multi-query — rewrite query N ways, union results
- Self-RAG / CRAG — retrieval as an optional tool
- GraphRAG — relationship-aware retrieval
- Context compression — LLMLingua, summary chains
- RBAC for RAG — per-tenant index vs metadata filter

#### RAG Evaluation ⬜
- Faithfulness: is answer grounded in retrieved context?
- Answer relevancy: does answer address the question?
- Context precision: are retrieved chunks relevant?
- Context recall: are all relevant chunks retrieved?
- Citation strategies: inline `[1]` vs structured objects

---

### Multi-Agent Systems
#### Agent Roles & Coordination ⬜
- Supervisor → assigns tasks to workers
- Planner → decomposes goal into subtasks
- Executor → executes individual steps
- Critic → evaluates outputs, triggers revision
- Specialist → domain-specific tool operators

#### MCP Protocol ⬜
- Client / Server / Host architecture
- Transport: stdio (local), SSE (web), HTTP (remote)
- `@mcp.tool()` — expose functions as tools
- `@mcp.resource()` — expose data as resources
- `@mcp.prompt()` — expose prompt templates
- Most candidates use MCP; fewer have *built* an MCP server

#### A2A Protocol ⬜
- Agent Card: agent capabilities, auth, skills declaration
- Task and TaskStatus lifecycle
- Push notifications for async tasks
- Agent discovery — service registry patterns

#### LangGraph ⬜
- Typed state schema (TypedDict)
- Nodes (functions) + Edges (transitions)
- Conditional edges — dynamic routing
- Checkpointing — persist state between steps
- Time-travel debugging — replay from any checkpoint

#### Failure Modes ⬜
- Infinite delegation loops — max-hop limit
- Context explosion across hops — "telephone game" degradation
- Conflicting plans — consensus mechanism or Critic agent
- Termination conditions: explicit stop, planner approval, max-iterations

---

### MLOps & Deployment
#### Docker ⬜
- Multi-stage builds — builder + runtime image
- Distroless base images — minimal attack surface
- `.dockerignore` — exclude secrets, caches
- Non-root user — principle of least privilege
- Layer caching — order instructions by change frequency

#### Kubernetes ⬜
- Deployment + Service + Ingress — the standard stack
- ConfigMaps + Secrets — externalise config; never bake API keys
- HPA — scale on CPU, custom metrics (queue depth, latency)
- StatefulSets — for memory-backed agent services
- Resource requests/limits — prevent noisy-neighbour issues

#### CI/CD for Agent Services ⬜
- Stages: lint → unit → integration → build → scan → push → deploy
- GitHub Actions vs Jenkins — trade-offs
- GitOps (ArgoCD/Flux) — declarative > imperative kubectl
- Canary releases — shadow, % shift, eval-metric rollback
- Prompt versioning — Git + prompt registry + CI gate

#### Observability ⬜
- Logs: structured JSON → ELK (Elasticsearch, Logstash, Kibana)
- Metrics: Prometheus scrape → Grafana dashboards; RED method
- Traces: OpenTelemetry → Jaeger/Tempo
- Agent-specific metrics: TTFT, end-to-end latency, token usage/cost, tool call count, retry rate
- LangSmith / Langfuse / Arize Phoenix — agent trace spans

---

### Evaluation & QA
#### Eval Tools ⬜
- Ragas — RAG-pipeline-focused; faithfulness/relevancy/precision/recall
- DeepEval — pytest-style assertions; G-Eval, hallucination metric
- TruLens — trace-instrumented feedback functions; groundedness/relevance/harmlessness
- Promptfoo — config-driven A/B across prompts/models; CI integration
- Guardrails AI — input/output validators, Pydantic-style rail spec

#### LLM-as-Judge ⬜
- When it works vs doesn't (position bias, length bias, self-preference)
- Calibration: human correlation studies
- Multi-judge ensembles for robustness

#### Eval-Driven Development ⬜
- Write the eval before writing the prompt
- Golden datasets — version-pinned, model-version-pinned
- Statistical significance — CI, paired tests; "3% better" may not be real

---

### Safety & Security
#### Prompt Injection ⬜
- Direct injection — adversarial user input
- Indirect injection — malicious content in retrieved docs
- Mitigations: instruction hierarchies, content sandboxing, output classifiers

#### Data & Privacy ⬜
- GDPR: data minimization, right to explanation, automated decision transparency
- EU AI Act: high-risk classification, documentation obligations
- PII detection and scrubbing in agent pipelines
- Data exfiltration through tools — egress filtering

#### Audit & Compliance ⬜
- Tamper-evident audit logs for every agent decision
- Tool call logging with input/output capture
- Trace retention policies

---

### Data Integration
#### SQL Agents ⬜
- Schema-aware prompting — table/column descriptions in context
- Query validation before execution (parse check + dry run)
- Sandboxed execution — read-only credentials, row limits
- Eval: execution-match (not string-match) against gold queries
- Spider / BIRD benchmarks

#### Kafka ⬜
- Topics, partitions, consumer groups
- Offset management — at-least-once vs exactly-once
- aiokafka for async Python consumers
- Agent-as-Kafka-consumer: event-driven triggers
- Dead-letter queues for failed agent tasks

---

### Frameworks
#### LangChain / LangGraph ⬜
- Runnables + LCEL pipe syntax
- Agents: tool decorators, memory, callbacks
- LangGraph: stateful multi-step graphs (see MAS section)
- Critique: abstraction overhead, version churn — have an opinion

#### LlamaIndex ⬜
- Query engines vs chat engines
- Index types: vector, summary, tree, keyword
- Routers — query routing to right index
- Response synthesizers — how results are composed

#### DSPy ⬜
- Signature — typed input/output declaration (no prompt strings)
- Module — composable unit (Predict, ChainOfThought, ReAct)
- Teleprompter/Optimizer — auto-compiles prompts from data
- Key insight: "program prompts, don't craft them"

#### CrewAI / AutoGen / Haystack ⬜
- CrewAI: role-based, lightweight; sequential + hierarchical processes
- AutoGen: `ConversableAgent`, `GroupChat`, `GroupChatManager`
- Haystack: traditional NLP pipeline shape; strong for document QA
