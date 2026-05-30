# Round 4: System Design — Multi-Agent at Automotive Scale — BMW AI Engineer
*Status: ⬜ Not Attempted | Duration: ~60 min | To run: `/interview-round BMW-AI-Engineer system-design`*

## Likely Design Problems (from JD hints)
1. **Warranty claim triage agent** — dealer emails → classify → route → respond; multi-agent with tool access to SAP
2. **Repair manual RAG** — 10 years of PDFs with diagrams; multi-modal; RBAC per dealer tier
3. **Logistics multi-agent** — Planner agent routes shipment; Executor checks supplier ETAs via ERP tools
4. **Quality defect pipeline** — production line sensor data → agent detects anomaly → escalation workflow
5. **Internal process automation** — employee query agent over HR/finance docs with policy guardrails

## Evaluation Rubric (what BMW cares about)
| Area | What they look for |
|------|--------------------|
| Requirements clarification | Scope, scale, users, SLAs, safety constraints |
| Agent architecture | Roles split clearly; tool interfaces defined; memory strategy |
| RAG design | Chunking rationale, index choice, hybrid retrieval, RBAC |
| Reliability | Budget controls, self-healing, idempotency, circuit breakers |
| Observability | What you'd instrument; TTFT, cost, error rate dashboards |
| Safety | Guardrails, audit trail, prompt injection defense |
| Deployment | Docker + K8s design; CI/CD gates; canary strategy |
| Trade-offs | Can articulate what you'd sacrifice and why |

## BMW-Specific Anchors to Weave In
- SAP integration → idempotent tool wrappers, transactional safety
- EU AI Act → every agent decision must be explainable and logged
- Manufacturing context → where agents must NOT act autonomously (safety-critical paths)
- CET timezone → 24/7 reliability not required, but on-call observability matters

## Results
*(Populated after `/interview-round BMW-AI-Engineer system-design` is run)*
