# Round 3: Technical — RAG + Evaluation — BMW AI Engineer
*Status: ⬜ Not Attempted | Duration: ~60 min | To run: `/interview-round BMW-AI-Engineer technical-2`*

## Focus Areas
- RAG pipeline end-to-end: chunking choices, embedding selection, hybrid retrieval, reranking
- Vector indexes: HNSW vs IVF-PQ — recall, latency, memory trade-offs
- Ragas metrics: faithfulness, answer relevancy, context precision/recall — definitions + gotchas
- LLM-as-judge: position bias, length bias, self-preference — how to calibrate
- DeepEval vs TruLens vs Promptfoo — when to reach for each
- Guardrails: input/output validation, PII scrubbing, jailbreak patterns
- RBAC for RAG: per-tenant isolation vs metadata filter — which fails silently?
- Context compression: LLMLingua, summary chains — when you're window-bound

## BMW-Specific Angles
- "Design RAG over BMW's 10 years of internal repair manuals (PDFs with diagrams)" — likely in this round
- Multi-modal RAG: handling diagrams, tables, CAD references in automotive docs
- RBAC in RAG: which BMW engineer can access which document tier?

## Expected Depth
Hands-on familiarity expected — "have you actually run Ragas on a real pipeline?" type probes.

## Results
*(Populated after `/interview-round BMW-AI-Engineer technical-2` is run)*
