# AI/ML Industry Primer — Concise, Interview-Focused Study Walkthrough

A dense companion to `content.md` and `reading_order.md`. Every topic uses the same five-lens template so you can ladder from "I've heard of it" to "I can defend it on a whiteboard at Anthropic."

---

## 0. How to use this primer

For each topic:

- **L1 — College basics:** the formal definition you'd see in a textbook.
- **L2 — Industry working knowledge:** what a competent IC needs to ship.
- **L3 — Senior/staff depth:** the tradeoffs, failure modes, and design judgment.
- **L4 — Interview grill:** the actual follow-up questions and how to answer.
- **CS cross-links:** other CS concepts this topic rests on or feeds into.
- **Verticals:** where it shows up — `[AI-lab | FAANG | Data | Fin | Consult]`.

Skip L1 if you already have it. The interview signal is in L3/L4.

---

## 1. CS prerequisites you must own (cross-cutting)

These show up in nearly every interview and underlie everything below. If any of these is shaky, fix it first.

| Area | Must-have | Where it bites |
|---|---|---|
| **Probability / stats** | Bayes rule; conditional expectation; CLT; bias-variance; hypothesis tests; CIs; bootstrap | A/B tests, eval, RL, generative models |
| **Linear algebra** | Matrix-vector products; eigendecomp; SVD; rank; norms; projections | Attention, embeddings, LoRA, quantization |
| **Calculus / optimization** | Gradients; chain rule; convexity; Lagrangians; KKT | Backprop, optimizers, constrained decoding |
| **Information theory** | Entropy; KL; mutual info; cross-entropy; perplexity | Loss design, distillation, eval |
| **Algorithms** | Big-O; DP; graphs; heaps; tries; suffix arrays | LeetCode, beam search, radix-tree cache, BPE |
| **Systems** | Cache hierarchy; concurrency; networking (TCP/RDMA); OS scheduling | Inference systems, parallelism, KV cache |
| **Distributed systems** | CAP; consensus; replication; back-pressure; idempotency | Training infra, vector DBs, serving |
| **Databases** | B-trees, LSM, indexing, query planning, transactions, isolation | Vector DBs, lakehouses, feature stores |
| **Software engineering** | Testing, types, error handling, observability, code review | Every job |

**Verticals:** all five.

---

## 2. The study walkthrough (4 phases × ~3 weeks each)

This converts `reading_order.md` into an action plan with interview practice baked in.

### Phase 1 — Foundations (Weeks 1–3)
- §1 Foundations + §2 Architectures + §3 Attention from `content.md`.
- **Build:** nanoGPT from scratch in one weekend. Implement attention without looking. Reproduce a tiny scaling-law curve on synthetic data.
- **Drill:** derive softmax-attention complexity; explain RoPE in 60 seconds; whiteboard a forward pass.
- **CS sharpen:** linear algebra, prob, info theory.

### Phase 2 — Training stack (Weeks 4–6)
- §4 Model families + §5 Pretraining + §6 Fine-tuning + §7 Alignment.
- **Build:** SFT a 1B model with LoRA on Axolotl/Unsloth; run DPO on it; eval against base on MT-Bench style task.
- **Drill:** PPO vs DPO vs GRPO derivations; Chinchilla optimal-compute math; tokenizer pitfalls.
- **CS sharpen:** optimization, distributed systems basics, MPI.

### Phase 3 — Inference & systems (Weeks 7–9)
- §15 Hardware + §8 Inference systems + §9 Optimization + §17 Profiling.
- **Build:** add KV paging or speculative decoding to your tinygrad engine; benchmark TTFT/ITL; quantize a model to INT4 (AWQ) and measure quality delta.
- **Drill:** roofline analysis on a specific kernel; speculative-decoding acceptance-rate math; design a multi-tenant LLM serving system.
- **CS sharpen:** systems, GPU programming, RDMA, OS scheduling.

### Phase 4 — Applied + Ops (Weeks 10–12)
- §11 RAG + §12 Agents + §13 Prompting + §14 Multimodal + §10 LLMOps + §16 Safety.
- **Build:** a non-trivial agent with MCP + custom eval harness + production tracing (Langfuse or your own).
- **Drill:** design a RAG-over-Confluence system; design an agent eval; design model rollout for a 10M-user app.
- **CS sharpen:** databases, distributed systems advanced, security.

Throughout: 1–2 LeetCode/day, 1 ML system design per week, 1 paper deep-read per week.

---

## 3. Section-by-section primer

### §1.1 Supervised / unsupervised / SSL
- **L1:** loss minimization over labeled / no-labels / pretext-task data.
- **L2:** know cross-entropy vs MSE choice; pretext tasks (MLM, NTP, contrastive); how MAE/BERT/GPT differ as SSL.
- **L3:** SSL transfer quality scales with model + data; choice of pretext determines what features emerge; collapse modes (BYOL vs SimCLR).
- **L4:** *"Why does GPT pretraining transfer to classification better than BERT for generative tasks but worse for embedding?"* — autoregressive objective biases representation toward sequence prediction, BERT's bidirectional context yields better static embeddings; cite E5 / BGE going back to encoder-style training.
- **CS:** information theory (cross-entropy as compression), prob (likelihood).
- **Verticals:** all.

### §1.2 Reinforcement learning
- **L1:** MDP (S, A, P, R, γ); maximize expected discounted return.
- **L2:** Q-learning vs policy gradient; on-policy vs off-policy; PPO clip, value/advantage (GAE).
- **L3:** sample efficiency; exploration-exploitation; reward shaping pitfalls; offline RL (CQL, IQL); credit assignment in sparse rewards.
- **L4:** *"Derive PPO's clipped objective and explain why clipping helps."* — limits ratio update to prevent destructive large policy changes; alternative is KL penalty (trust region). *"Why does REINFORCE have high variance and how does GAE reduce it?"* — baseline + bias-variance tradeoff via λ.
- **CS:** dynamic programming (Bellman), prob, optimization.
- **Verticals:** AI-lab (RLHF), Fin (algo trading via RL).

### §1.3 Online / active / continual / federated
- **L1:** sequential decisions over time / select-what-to-label / non-stationary tasks / decentralized training.
- **L2:** regret in online; uncertainty sampling in active; EWC for continual; FedAvg/FedProx; secure aggregation.
- **L3:** continual ≠ online; catastrophic forgetting mitigations; federated comm costs > compute costs; DP-federated.
- **L4:** *"Your customer wants on-device fine-tuning that respects privacy — design it."* — LoRA on-device + federated aggregation + DP noise + secure agg.
- **Verticals:** Fin (privacy), FAANG (recsys online), Consult (cross-org).

### §1.4 Meta / transfer / few/zero-shot
- **L1:** learning to learn / reusing learned features / generalize from few examples.
- **L2:** MAML's bilevel optimization; prototypical nets; in-context learning as implicit Bayesian inference (Xie 2022).
- **L3:** when fine-tuning beats few-shot ICL (>~50 examples), when ICL wins (rapid prototype, frequent task changes); cost analysis.
- **L4:** *"Why does ICL work?"* — implicit Bayes / induction heads / pretraining distribution coverage; cite Olsson "induction heads" or Garg "what can transformers learn in-context".
- **CS:** Bayesian inference.
- **Verticals:** all.

### §1.5 Contrastive / representation
- **L1:** pull similar pairs together, push dissimilar apart in embedding space.
- **L2:** InfoNCE; positive pair construction (augmentation in vision, query-document in retrieval); hard-negative mining.
- **L3:** uniformity/alignment metrics (Wang & Isola); collapse without negatives → BYOL/DINO solutions (stop-gradient, EMA); temperature is critical.
- **L4:** *"Why does CLIP scale and what would you change?"* — scale laws on data, large negatives → big batch; followups: SigLIP avoids softmax across batch.
- **CS:** metric learning, IR (BM25 link).
- **Verticals:** AI-lab, FAANG (search/recs), Data (vector DB).

### §1.6 Probabilistic / Bayesian / causal
- **L1:** parameters as RVs / posterior beliefs; cause vs correlation.
- **L2:** ELBO derivation; HMC vs NUTS; Pearl's three-step ladder (assoc, intervention, counterfactual); do-operator; backdoor adjustment.
- **L3:** when to go full Bayes (small data, decision-critical, uncertainty needed); causal inference for A/B with confounding; IV in observational.
- **L4:** *"Recommender shows correlation between feature X and conversion — would you ship it?"* — confounders, Simpson's, randomization argument; mention DoWhy / EconML.
- **CS:** stats, graph theory (DAGs), optimization.
- **Verticals:** Fin, Consult, FAANG (causal A/B), AI-lab (interp uses causal scrubbing).

### §1.7 Optimization theory
- **L1:** minimize a function on parameters.
- **L2:** SGD + momentum; AdamW math; LR schedules (cosine, WSD); gradient clipping rationale.
- **L3:** Adam's second-moment estimation; when Lion/Sophia/Muon win (memory, scale); preconditioned methods (Shampoo, K-FAC); μP/mu-transfer for HP transfer.
- **L4:** *"Why AdamW not Adam?"* — decoupled weight decay; *"Walk through Muon and why it works on hidden weights."* — orthogonalized momentum, matches loss-landscape geometry.
- **CS:** numerical analysis, convex opt.
- **Verticals:** AI-lab, FAANG.

### §1.8 Generalization, scaling laws, NTK
- **L1:** train loss ≠ test loss; loss decreases as power-law in compute/data/params.
- **L2:** Kaplan vs Chinchilla; compute-optimal ratio (20 tokens/param Chinchilla, much higher now in over-train regimes); double descent; NTK lazy regime.
- **L3:** inference-aware scaling (longer-trained smaller models for serving); data-constrained scaling (Muennighoff); transfer of scaling laws across modalities.
- **L4:** *"You have $1M of GPU budget — pick model size and data tokens."* — derive from Chinchilla, then adjust for inference budget, then for data availability.
- **CS:** stat learning theory.
- **Verticals:** AI-lab.

---

### §2.1 Classical NNs (MLP/CNN/RNN/LSTM)
- **L2:** universal approximation; CNN inductive biases (locality, translation); RNN BPTT vanishing/exploding; LSTM gates.
- **L4:** *"Why did Transformers replace RNNs for NLP?"* — parallelization across sequence + better long-range; not always — RNN/SSMs winning back for long context.
- **CS:** signal processing.
- **Verticals:** all (legacy + recsys still CNN/MLP).

### §2.2 Transformer
- **L1:** encoder/decoder stacks of self-attention + FFN with residuals + layer norm.
- **L2:** "Attention is All You Need" line by line; layer norm pre vs post; positional encodings (sinusoidal, learned, RoPE); residual stream interpretation; encoder-only (BERT) vs decoder-only (GPT) vs enc-dec (T5).
- **L3:** why decoder-only dominates (training efficiency, single objective, scales cleanly); why GPT-3 was the inflection; why some still use enc-dec (T5 for code, Whisper for ASR); norm placement affects gradient flow.
- **L4:** *"Implement scaled-dot-product attention in 10 lines."* — must include √d, mask, softmax dim, dropout. *"What's the role of LayerNorm and why pre-norm?"* — gradient stability, depth scaling.
- **CS:** linear algebra, parallelism.
- **Verticals:** all.

### §2.3 Mixture of Experts (MoE)
- **L2:** top-k routing, capacity factor, load balancing aux loss; expert vs token choice.
- **L3:** why MoE wins (sparse FLOPs, more params at same compute); routing collapse failure mode; DeepSeekMoE shared experts; serving challenges (expert parallelism, all-to-all comm).
- **L4:** *"Design inference for a 670B MoE with 37B active."* — expert parallelism, all-to-all routing on NVLink, prefill vs decode different optimal sharding, DeepSeek-V3 style aux-loss-free balancing.
- **CS:** routing, distributed systems.
- **Verticals:** AI-lab (frontier models), FAANG.

### §2.4 SSMs (Mamba, S4, Mamba-2, Jamba)
- **L2:** linear-time recurrence with structured state; selective state via input-dependent params; HiPPO motivation.
- **L3:** why O(L) memory matters for long-context; hybrid SSM+attention models (Jamba, Zamba) hedge against SSM weaknesses (in-context retrieval).
- **L4:** *"Why does Mamba struggle with copying tasks?"* — fixed-size state can't perfectly recall arbitrary tokens; attention's KV cache is per-token, scaling-aware.
- **CS:** control theory, signal processing.
- **Verticals:** AI-lab.

### §2.5 Diffusion / GAN / VAE / Flows
- **L2:** DDPM forward noise + reverse denoise; CFG; flow matching as a unifying view; VAE ELBO; coupling layers.
- **L3:** why flow matching is taking over (no SDE, deterministic, faster); image vs video diffusion challenges (temporal consistency); rectified flow (FLUX, Stable Diffusion 3).
- **L4:** *"Derive the DDPM loss from the ELBO."* — be ready. *"Why is GAN training unstable?"* — minimax + non-convergent dynamics; WGAN/Wasserstein distance fixes.
- **CS:** SDEs, optimization (minimax).
- **Verticals:** AI-lab (multimodal teams), FAANG (recsys gen).

### §2.6 GNNs
- **L2:** message passing; GCN, GAT, GraphSAGE; inductive vs transductive.
- **L3:** over-smoothing fix (residuals, attention); heterogeneous graphs; scalability (mini-batch sampling); GNN for KG-RAG.
- **L4:** *"Why hasn't GNN replaced MLP for tabular data?"* — implicit feature interactions vs MLP's universal approx; only wins with strong relational structure.
- **Verticals:** Fin (fraud), FAANG (recs), Data (KGs).

### §2.7 Multimodal architectures
- **L2:** CLIP contrastive; LLaVA simple projector; Flamingo cross-attn with frozen LM; Qwen-VL dynamic resolution.
- **L3:** early- vs late-fusion; image token budget vs context; native multimodal training (Gemini, GPT-4o) vs bolt-on.
- **L4:** *"VLM struggles with high-res documents — fix it."* — dynamic resolution + multi-scale tokens + OCR augmentation + specialized FT (Docling/Nougat).
- **Verticals:** AI-lab, FAANG, Consult (document-heavy industries).

---

### §3 Attention (the most-grilled section in AI-lab interviews)

### §3.1 MHA / MQA / GQA / MLA
- **L1:** projections Q, K, V, dot product, softmax, weighted V.
- **L2:** MHA = H parallel heads; MQA = 1 KV head shared; GQA = G KV head groups; MLA = low-rank KV compression (DeepSeek). KV cache size: 2 · L · H · d · bytes per layer.
- **L3:** MQA hurts quality slightly, saves 8x KV memory; GQA is the production default; MLA compresses 90%+ via shared-up-projection trick + decoupled RoPE; tradeoff in TP sharding.
- **L4:** *"Compute KV cache size for Llama 3 70B at 128K context, BF16."* — derive H_kv, n_layers, d_head, ×2 (K+V), ×bytes, ×ctx, ×batch. *"How does MLA recover quality lost vs MQA?"* — low-rank decomposition retains more info than head-sharing alone; decoupled position handling.
- **CS:** linear algebra, memory hierarchy.
- **Verticals:** AI-lab, FAANG inference teams.

### §3.2 Sparse / sliding-window / linear
- **L2:** Longformer global+local; BigBird random+local+global; sliding window (Mistral); Performer FAVOR+; Linformer low-rank.
- **L3:** when window attention is enough (most decoder workloads with short query / long context with prefix cache); linear attn quality gap; RWKV as RNN reframe.
- **L4:** *"Mistral uses sliding window — implications for serving?"* — KV cache cap at window size; rolling buffer; prefix cache still valid.
- **Verticals:** AI-lab inference.

### §3.3 FlashAttention v1/v2/v3
- **L2:** IO-aware tiling; online softmax; v2 work partitioning; v3 H100 async + FP8.
- **L3:** why naive attention is memory-bound (softmax requires N × N matrix in HBM); FA's tile size and softmax recomputation in backward; v3 producer-consumer pipelining.
- **L4:** *"Walk through online softmax."* — running max + running sum, rescale on each block; *"Why does FA help at large N specifically?"* — kills HBM traffic; *"FP8 in FA3 — what breaks?"* — calibration of K/V scales, softmax precision.
- **CS:** memory hierarchy, GPU programming.
- **Verticals:** AI-lab, FAANG.

### §3.4 Paged / Ring / Blockwise / Tree / Radix
- **L2:** vLLM PagedAttention block tables (KV virtual memory); Ring Attention for cross-device long context; SGLang RadixAttention for prefix sharing across requests.
- **L3:** fragmentation in non-paged systems; block size tuning; ring vs Ulysses CP tradeoffs.
- **L4:** *"Design KV cache management for a multi-tenant serving system."* — page-based + LRU eviction + per-tenant quotas + prefix dedup via radix.
- **CS:** OS virtual memory, distributed systems.
- **Verticals:** AI-lab, FAANG, Data (Snowflake's Cortex).

### §3.5 RoPE / ALiBi / YaRN / NTK-aware
- **L2:** RoPE rotates QK pairs by position-dependent angles; ALiBi adds linear bias; YaRN rescales for context extension.
- **L3:** why RoPE wins (relative + extrapolatable + math-clean); RoPE base θ → context length tradeoff; NTK-aware base scaling; LongRoPE per-dimension scaling.
- **L4:** *"Extend a 4K-context model to 128K."* — continued pretraining + YaRN/NTK + long-context SFT + needle-in-haystack eval.
- **CS:** complex analysis, signal processing.
- **Verticals:** AI-lab.

### §3.6 Attention sinks, differential attention
- **L2:** StreamingLLM observation: first tokens accumulate attention mass; removing them breaks model; keep N sink tokens + sliding window.
- **L4:** *"Why do sinks exist?"* — softmax always sums to 1; model needs somewhere to send "no useful attention" mass; LLMs learn first tokens as sinks.
- **Verticals:** AI-lab.

---

### §4 LLMs — models & families

### §4.1 The model landscape
- **L2:** know architecture/quirks of: GPT-4o/4.1/o-series; Claude 3.5/4 family; Gemini 1.5/2; Llama 3.x/4; Qwen 2.5/3; DeepSeek V3/R1; Mistral/Mixtral; Phi/Gemma.
- **L3:** for each: license, context length, MoE/dense, native multimodal y/n, reasoning variant. Track release cadence.
- **L4:** *"Pick a model for $X use case."* — be ready with cost, latency, quality, license, and trust tradeoffs; e.g., regulated industry → on-prem capable (Llama, Qwen, DeepSeek) over API-only.
- **Verticals:** all (consultative product decisions).

### §4.2 Base vs instruct vs reasoning
- **L2:** base = PT-only; instruct = SFT + DPO/RLHF; reasoning = + RLVR/GRPO with verifiable rewards.
- **L4:** *"When use base?"* — continued PT, domain-specific synthetic data, custom alignment recipes.

### §4.3 Reasoning models
- **L2:** "thinking" tokens; o1/o3, R1, QwQ; longer CoT at inference; trade latency for accuracy.
- **L3:** test-time compute scaling laws; cost-quality curves; when reasoning hurts (simple lookup tasks).
- **L4:** *"Why does CoT scaling work?"* — implicit search over chains of inference; PRM-guided or verifier-guided variants.
- **Verticals:** AI-lab.

### §4.4 SLMs / edge
- **L2:** Phi-3/4, Gemma-3, SmolLM, Qwen-0.5/1.5B; on-device via MLX/CoreML/ONNX.
- **L4:** *"On-device LLM for an iPhone app — design constraints?"* — quantized 3–7B, NPU mapping, KV size, battery, OTA model updates.
- **Verticals:** FAANG (mobile), Consult (privacy clients).

---

### §5 Pretraining

### §5.1 Data curation, dedup
- **L2:** Common Crawl → extraction → filter → dedup → mix; MinHash-LSH near-dedup; SemDeDup; benchmark contamination removal.
- **L3:** quality-classifier pipelines (FineWeb-Edu); domain mix as HP; data attribution; legal/copyright considerations.
- **L4:** *"How do you avoid benchmark contamination?"* — n-gram blocklist, perplexity-on-test detection, canary insertion, dedup-against-eval-set.
- **CS:** distributed systems (PB-scale data pipelines), hashing.
- **Verticals:** AI-lab, Data.

### §5.2 Tokenization
- **L2:** BPE algorithm; SentencePiece BPE/Unigram; byte-level BPE; vocabulary size 32K–256K range.
- **L3:** multilingual fairness (token cost varies 5× across languages); code vs natural language vocab tradeoffs; tokenizer changes invalidate all checkpoints.
- **L4:** *"You're adding a new language to a Llama model — replace the tokenizer?"* — extend with new BPE merges + init new embeddings + continued PT; full replace nukes all weights' alignment.
- **CS:** data structures (tries, merge ops).
- **Verticals:** AI-lab, Consult (localization).

### §5.3 Data mixing, DoReMi
- **L2:** mix proportions as HPs; DoReMi uses group DRO to learn weights from a reference.
- **L4:** *"Why is the math/code mix usually heavier than natural ratio?"* — better reasoning transfer; "high-quality" annealing in late stages.
- **Verticals:** AI-lab.

### §5.4 Synthetic data, distillation PT
- **L2:** Self-Instruct, Evol-Instruct, Phi-style textbook synthesis.
- **L3:** mode collapse risk; eval contamination; teacher-student gap.
- **L4:** *"Phi punches above its weight — why?"* — curated high-quality synthetic data; risk: benchmark overfit.
- **Verticals:** AI-lab.

### §5.5 Long-context PT, context extension
- **L2:** YaRN, position interpolation, staged extension.
- **L4:** *"Pass needle test but fail real long-context tasks — why?"* — needle is easy; realistic tasks need multi-hop reasoning over context; eval with RULER, LongBench.

### §5.6 Annealing, LR schedules
- **L2:** cosine vs WSD; high-quality data in decay phase.
- **L4:** *"You're 80% through PT — change data mix?"* — yes; anneal phase is where quality is built; cite Llama 3 / DCLM.

### §5.7 Scaling laws
- **L2:** Kaplan 2020, Chinchilla 2022 (20 tok/param), inference-aware (over-training small).
- **L4:** *"Llama 3 8B trained on 15T tokens — Chinchilla says 160B. Why?"* — inference economics; smaller model serves cheaper, over-train pays for itself in production lifetime.

### §5.8 Optimizers
- **L2:** AdamW default; Lion (sign-based, memory-efficient); Sophia (Hessian-diag); Shampoo (preconditioned); Muon (orthogonalized momentum for matrices).
- **L4:** *"Walk through Muon."* — Newton-Schulz orthogonalize momentum, apply to 2D matrices, leave embeddings/LN to AdamW; cite Keller Jordan, modded-nanogpt records.

---

### §6 Fine-tuning

### §6.1 FFT / SFT / instruction tuning
- **L2:** SFT = next-token loss on (instruction, response); chat templates; loss mask on assistant tokens only; data packing.
- **L4:** *"Why does loss masking matter?"* — training on instruction tokens biases toward repeating user input; *"What's a chat template?"* — model-specific tokens (`<|im_start|>`, `<|user|>`) that frame turns.

### §6.2 LoRA / QLoRA / DoRA
- **L2:** LoRA: ΔW = αBA, rank r, target attn projections; QLoRA: 4-bit base + LoRA on top; DoRA: decompose into magnitude + direction.
- **L3:** rank r typically 8–64; α/r ratio matters; rsLoRA fixes scaling at high rank; QLoRA's NF4 / double-quant; merging LoRA back (cheap) vs adapter swapping at serve time (S-LoRA, Punica).
- **L4:** *"LoRA vs full FT quality gap and when it disappears?"* — gap shrinks with larger rank, more layers targeted, longer training; for catastrophic-forgetting-sensitive tasks LoRA actually wins.
- **CS:** linear algebra (low-rank approx, SVD).
- **Verticals:** all.

### §6.3 Adapters, prefix/prompt tuning, IA³, BitFit
- **L2:** parameter-count comparison; insertion points.
- **L4:** *"Multi-tenant serving with per-customer adapters — design."* — S-LoRA or Punica style: hot-swap LoRA at decode time, batch heterogeneous adapters.
- **Verticals:** FAANG (multi-tenant LLM).

### §6.4 Distillation FT
- **L2:** response (logit), feature, attention-based; reverse vs forward KL.
- **L3:** on-policy distillation > off-policy at quality; reverse KL → mode-seeking (sharp), forward KL → mode-covering (diverse).
- **L4:** *"Distill 70B → 7B — choose loss."* — start with response KL, optionally add intermediate feature matching; iterate on-policy.

### §6.5 Model merging
- **L2:** averaging weights; task arithmetic (W_finetuned − W_base = task vector); TIES, DARE, SLERP.
- **L4:** *"Merge two SFT models trained on different domains — when does it work?"* — same base required; conflicting task vectors degrade; TIES trims and resolves sign conflicts.

---

### §7 Alignment & RL for LLMs (heavy AI-lab grill area)

### §7.1 RLHF (PPO)
- **L2:** loop: SFT → reward model → PPO with KL penalty to SFT.
- **L3:** reward model trained with Bradley-Terry on preference pairs; KL coefficient tuning; reference model memory cost (2x); reward hacking (length bias, sycophancy).
- **L4:** *"PPO has 4 models in memory — name them."* — actor, critic, reward model, frozen reference. *"How do you detect reward hacking?"* — pairwise human eval, OOD probes, length/style controls.
- **CS:** RL (PPO, GAE).
- **Verticals:** AI-lab.

### §7.2 DPO & family
- **L2:** DPO closed-form: reward = β · log(π/π_ref); maximize logistic loss over preference pairs.
- **L3:** DPO derivation from RLHF objective; IPO regularization; KTO unpaired; ORPO no-reference-model; SimPO length-normalized.
- **L4:** *"Derive DPO objective starting from KL-regularized RL."* — be ready: reward = optimal policy's log-ratio, plug into BT preference model. *"Why does DPO sometimes underperform PPO?"* — off-policy, no exploration, sensitive to data distribution.

### §7.3 GRPO / RLOO
- **L2:** no value model; baseline = group mean of N samples per prompt; standardize advantages.
- **L4:** *"DeepSeek's R1 uses GRPO — why not PPO?"* — sample efficiency on verifiable-reward tasks; no need for accurate value estimator when reward is binary.

### §7.4 PRM vs ORM
- **L2:** PRM = per-step rewards; ORM = final-answer only.
- **L4:** *"When does PRM help?"* — long-CoT math/code; cite "Let's Verify Step by Step".

### §7.5 RLAIF, Constitutional AI
- **L2:** AI preference labels; CAI critique-revise loop with a constitution.
- **L4:** *"Trust AI-generated preferences over human?"* — scale + speed wins for breadth; humans still needed for novel-harm coverage; mix recommended.

### §7.6 Verifiable rewards (RLVR)
- **L2:** code unit tests, math answer checking, format checks.
- **L4:** *"Design RLVR for a new domain (e.g., SQL writing)."* — exec-based reward (correctness + perf), step-level test partials, length normalization.

### §7.7 Safety tuning
- **L2:** harmlessness data; refusal training; over-refusal mitigation.
- **L4:** *"Reduce over-refusal without losing safety."* — XSTest-style false-positive probes; targeted-refusal SFT; uplift evals.

---

### §8 Inference systems (system-design heavyweight)

### §8.1 Framework landscape
- **L2:** vLLM (ergonomic, PagedAttention origin); SGLang (radix cache + structured gen); TRT-LLM (NVIDIA perf max); TGI (HF mature); LMDeploy (multi-backend); llama.cpp (CPU/edge); MLC-LLM (cross-platform); MAX (Modular).
- **L4:** *"Pick a serving stack for 100K QPS chat with 50% prefix overlap."* — SGLang for radix-tree prefix sharing OR vLLM with APC; multi-replica + per-replica continuous batching.
- **Verticals:** AI-lab, FAANG, Consult.

### §8.2 Batching
- **L2:** static (wait for batch), dynamic (timeout-based), continuous (per-iter scheduling, Orca).
- **L3:** continuous batching mixes prefill and decode in same step; chunked prefill limits prefill-induced ITL spikes.
- **L4:** *"Why does continuous batching dominate?"* — variable-length completions; one slow request shouldn't stall fast ones.

### §8.3 Prefill-decode disaggregation
- **L2:** prefill is compute-bound, decode is memory-bound; running together causes interference.
- **L3:** Splitwise/DistServe separate GPUs; Mooncake separates further with global KV pool; KV transfer over NVLink/RDMA; cross-node latency vs colocated benefit.
- **L4:** *"Design disagg serving for a 405B model."* — TP-sharded prefill cluster + TP-sharded decode cluster; KV transfer via NCCL or custom RDMA; admission control on TTFT SLO.

### §8.4 KV cache: paging, offload, hierarchical
- **L2:** block-table mapping (vLLM); CPU/NVMe offload; prefix cache as L2.
- **L3:** offload bandwidth math (PCIe 5: 64 GB/s; HBM3: ~3 TB/s); only offload prefill-only or cold prefixes.
- **L4:** *"KV cache OOM — what do you do?"* — quantize KV to FP8/INT4, evict by LRU or attention-score (H2O), offload to CPU/NVMe, paged scheduling.

### §8.5 Prefix / radix caching
- **L2:** dedup shared prefixes across requests; APC (vLLM), radix tree (SGLang).
- **L4:** *"Multi-tenant prefix sharing — privacy risk?"* — yes, side-channel via timing; isolate per-tenant or per-trust-group.

### §8.6 KV compression
- **L2:** KIVI 2-bit KV; KVQuant; H2O eviction; FP8 KV.
- **L4:** *"INT4 KV quality drop — fix it."* — per-channel scaling + outlier handling; recent: SnapKV.

### §8.7 Scheduling, SLO-aware, multi-tenant
- **L2:** FCFS / SJF / EDF / priority + preemption.
- **L4:** *"Two tenants — a paying chat customer needs <500ms TTFT, batch job is OK with 1min — design scheduler."* — priority queues + preemption + fairness floor + admission control on chat queue depth.

### §8.8 Disaggregated serving (transport)
- **L2:** NCCL collectives; RDMA; GPUDirect; UCC/UCX.
- **L4:** *"KV transfer overhead — when does disagg lose?"* — small models, short prefills, cross-rack latency > prefill saving.

### §8.9 Learned schedulers
- **L4:** *"RL-based scheduling — train how?"* — offline RL on traces; reward = SLO satisfaction; cite vTrain/Sarathi.

---

### §9 Inference optimization (the deep-dive section for systems-heavy interviews)

### §9a Speculative decoding
- **L2:** draft model proposes K tokens, target verifies in one forward pass; accept matching prefix; key: cheap draft.
- **L3:** acceptance rate ∝ alignment of draft and target; Medusa adds multiple prediction heads on frozen base; EAGLE drafts feature vectors (not tokens) with autoregressive tree; lookahead uses Jacobi iteration (n-gram lookup).
- **L4:** *"Derive expected speedup from acceptance rate α and draft cost c."* — speedup = (1 − α^{K+1})/((1−α)(c + 1)); *"Why does EAGLE-2/3 win?"* — dynamic tree expansion + draft tuned on hidden states.
- **CS:** rejection sampling, probability.
- **Verticals:** AI-lab.

### §9b Quantization
- **L2:** PTQ (no retrain), QAT (with retrain); GPTQ (Hessian-based), AWQ (activation-aware), SmoothQuant (scale activations into weights), HQQ (calibration-free), AQLM (vector quantization).
- **L3:** weight-only good to INT4; W+A quant needs SmoothQuant; FP8 E4M3/E5M2 tradeoffs; outlier channels are the main failure mode; KV quantization (KIVI, FP8).
- **L4:** *"Why does AWQ outperform GPTQ?"* — protects salient weights via per-channel scaling, no second-order info needed. *"Calibration set choice impact?"* — distribution matters; bad calibration = bad outliers.
- **CS:** numerical analysis, fixed-point arithmetic.

### §9c Sparsity & pruning
- **L2:** 2:4 structured sparsity on Ampere+; SparseGPT, Wanda; activation sparsity (Deja Vu, PowerInfer); structured pruning (ShearedLlama, LLM-Pruner).
- **L3:** unstructured pruning easier to find but harder to accelerate; activation sparsity exploits run-time dead neurons; Mixture-of-Depths skips layers per-token.
- **L4:** *"INT4 + 2:4 sparsity stack — pitfalls?"* — quality compounds, need careful calibration; tensor-core 2:4 path support.

### §9d Compilation & kernels
- **L2:** torch.compile (Dynamo+Inductor), Triton, CUTLASS, CUDA Graphs, TRT.
- **L3:** when to write a kernel (memory-bound, novel op, fusion opportunity); CUDA Graphs for low-latency decode (replay launch overhead); persistent kernels.
- **L4:** *"Why is FA a Triton kernel and not just PyTorch?"* — fusion of QK^T, scale, mask, softmax, AV in one kernel = no HBM roundtrip; *"Write a fused RMSNorm in Triton."* — be ready.
- **CS:** compilers, GPU programming.
- **Verticals:** AI-lab, FAANG infra.

### §9e Parallelism
- **L2:** DP, TP (Megatron row/col), PP (1F1B, interleaved, zero-bubble), SP (sequence), CP (Ring/Ulysses), EP (MoE).
- **L3:** when to choose what — TP for fitting one model on one node, PP for cross-node, EP for MoE, DP for throughput, ZeRO for memory; bubble math in PP.
- **L4:** *"You have 64 H100s — train a 70B model — design 3D parallelism."* — TP=8 (intra-node NVLink), PP=4 (cross-node), DP=2; ZeRO-1 on DP; activation checkpointing; estimate global batch size.
- **CS:** distributed systems, MPI, NCCL.
- **Verticals:** AI-lab, FAANG.

### §9f Other inference opts
- **L2:** constrained decoding (outlines FSM, XGrammar pushdown, lm-format-enforcer); samplers (top-k, top-p, min-p, typical, Mirostat, DRY, XTC); FlashDecoding for low-batch decode.
- **L4:** *"Why min-p over top-p for creative generation?"* — adaptive cutoff based on top-1 prob; top-p clips low-confidence too aggressively.

---

### §10 LLMOps / MLOps (most-asked at FAANG, Data, Consult)

### §10a Experiment & model management
- **L2:** W&B/MLflow for runs, sweeps, artifacts; model registry stages; HF Hub patterns.
- **L4:** *"Design model lifecycle for 50 fine-tunes/week."* — registry + auto-eval gate + canary; tag-based rollback.

### §10b Pipelines & orchestration
- **L2:** Airflow/Dagster/Prefect (DAG); Kubeflow/Flyte/Metaflow (ML-native); Ray for distributed Python.
- **L4:** *"Train pipeline retries flaky GPU jobs — design."* — checkpoint frequency, idempotent stages, exponential backoff, failure quarantine.
- **CS:** distributed systems.

### §10c Training infra
- **L2:** DeepSpeed (ZeRO configs); Megatron-Core; NeMo; Axolotl/LLaMA-Factory/Unsloth defaults.
- **L3:** Slurm vs Kubernetes (KubeRay, training-operator); gang scheduling; async checkpointing (DCP); elastic training (TorchElastic).
- **L4:** *"Multi-node FT job dies at hour 6 — how to make it survive?"* — frequent async checkpoints to fast NVMe + S3, elastic restart, checkpoint validation.

### §10d Data & feature infra
- **L2:** feature store (Feast online/offline parity); vector DB choice; data lakehouse (Delta/Iceberg/Hudi); streaming (Kafka/Flink).
- **L3:** training/serving skew root cause: feature drift between offline join and online lookup; online-feature freshness SLA.
- **L4:** *"Pick vector DB for 10B vectors with <50ms p99."* — DiskANN or IVF-PQ on Milvus/Vespa; HNSW too memory-hungry at this scale; partition + replica strategy.
- **CS:** databases, distributed systems, streaming.
- **Verticals:** Data (Databricks/Snowflake big), FAANG.

### §10e Deployment & serving ops
- **L2:** canary, blue/green, shadow; model gateway (LiteLLM/Portkey/OpenRouter); token-level cost attribution.
- **L4:** *"Roll out a new LLM version with 1% regression risk acceptable."* — shadow traffic + side-by-side eval + 1%→5%→25% canary + auto-rollback on quality metric + SLO breach trigger.

### §10f Monitoring & observability
- **L2:** LangSmith/Langfuse/Phoenix/Helicone/Braintrust for traces; OTel GenAI conventions; drift detection (KS, MMD, embedding-space).
- **L3:** trace structure (span/generation/score); cost attribution by feature; hallucination scoring (groundedness with NLI or LLM-judge); latency dashboards (TTFT/ITL/throughput).
- **L4:** *"Your agent is failing 5% of the time silently — how do you find why?"* — trace-level instrumentation; cluster failed traces by embedding; failure-mode taxonomy; tie to upstream prompt/model version.
- **CS:** observability, streaming analytics.
- **Verticals:** all (Consult: client trust, FAANG: scale, AI-lab: research velocity).

### §10g Evaluation
- **L2:** benchmark menu (MMLU-Pro, GPQA, HumanEval, LiveCodeBench, SWE-bench, GAIA, τ-bench, BFCL, IFEval); LLM-as-judge; eval harnesses (lm-eval-harness, Inspect, Promptfoo, RAGAS).
- **L3:** contamination detection; eval variance (run 5×, report CI); pairwise > pointwise for subjective; live arenas vs static benches.
- **L4:** *"Is model A 0.5% better than model B real?"* — bootstrap CIs; multi-seed; LMSYS arena methodology (BT/Elo); paired comparison test.
- **CS:** stats (hypothesis testing, bootstrap), experimental design.
- **Verticals:** all.

### §10h Governance, compliance
- **L2:** model cards / data sheets / system cards; PII redaction; EU AI Act tiers; NIST AI RMF; ISO 42001; DPDP Act 2023.
- **L4:** *"Compliance for a fintech LLM in EU + India."* — risk classification under AI Act (high-risk if credit decisioning), DPDP consent + purpose limitation, model card with bias eval, audit log retention.
- **Verticals:** Fin, Consult, big-co AI-lab.

---

### §11 Retrieval & RAG (heavy at FAANG, Data, Consult)

### §11.1 Sparse retrieval (BM25, SPLADE, uniCOIL)
- **L2:** BM25 = tf × idf × length-norm; inverted index.
- **L4:** *"BM25 vs dense — pick one?"* — neither; hybrid + RRF wins almost always; BM25 better for exact-match / rare terms / out-of-domain.
- **CS:** IR fundamentals.

### §11.2 Dense retrieval
- **L2:** bi-encoder + in-batch negatives + hard negatives; modern embedders (BGE, E5, GTE, Voyage, Cohere, OpenAI text-embedding-3-large).
- **L3:** Matryoshka embeddings; embedding model freshness (frequent updates); MTEB benchmark; multilingual considerations.
- **L4:** *"Production embedding choice criteria?"* — domain match, dimension cost (storage + compute), latency, license, MTEB performance on your task type.

### §11.3 ColBERT / late interaction
- **L2:** per-token embeddings; MaxSim scoring; ColBERTv2 residual compression.
- **L4:** *"Why doesn't everyone use ColBERT?"* — index size ~30× higher; PLAID engineering required; bi-encoder + reranker usually wins on cost.

### §11.4 Hybrid + reranking
- **L2:** RRF fusion; cross-encoder rerankers (BGE, Cohere Rerank, Jina).
- **L4:** *"Design RAG with budget $0.001/query and 200ms p95."* — BM25 + small dense retriever → top-100 → reranker → top-5; cache embeddings.

### §11.5 Query rewriting, HyDE, multi-query
- **L2:** HyDE generates hypothetical answer, embed it for retrieval; multi-query expansion.
- **L4:** *"When does HyDE hurt?"* — when LLM hallucinates wrong domain context; in-domain queries.

### §11.6 Chunking
- **L2:** fixed-size, recursive, semantic, parent-child, late-chunking.
- **L4:** *"Chunk size choice?"* — context window vs precision; ~512 tokens typical with 50–100 overlap; semantic chunking for heterogeneous docs.

### §11.7 GraphRAG, KG-augmented
- **L2:** Microsoft GraphRAG (entity-relation extraction + Leiden communities + summaries); LLM as KG-builder.
- **L4:** *"When is GraphRAG worth the cost?"* — multi-hop QA over interconnected docs (M&A, legal); not for simple lookup.
- **Verticals:** Consult, Fin (legal/regulatory).

### §11.8 Self-RAG, CRAG
- **L2:** retrieval as a learned action; relevance scoring controls whether to use retrieved doc.

### §11.9 Long-context vs RAG
- **L2:** "just stuff it in 1M context" tempts but fails on recall + cost; prompt caching mitigates cost.
- **L4:** *"1M-context model is here — is RAG dead?"* — no; cost, latency, freshness, precision still favor RAG; long-context helps for coherent docs.

### §11.10 Eval
- **L2:** recall@k, nDCG, MRR, hit-rate; BEIR; RAGAS for end-to-end (faithfulness, relevance, context recall/precision).
- **L4:** *"Recall is 95% but answer quality drops — diagnose."* — context too long → reasoning suffers; rerank failing; prompt template issue; cite "Lost in the Middle."

### §11.11 ANN indexes (HNSW, IVF, IVF-PQ, DiskANN, ScaNN)
- **L3:** HNSW: high recall, RAM-hungry; IVF-PQ: memory-efficient with PQ compression; DiskANN: SSD-resident, billion scale; ScaNN: anisotropic loss.
- **L4:** *"100M vectors, 16GB RAM, p99 50ms — pick index."* — DiskANN or IVF-PQ; HNSW won't fit.
- **CS:** approximate algorithms, data structures.

### §11.12 Vector DB CI/CD
- **L2:** recall gating on eval set; blue/green index cutover.

---

### §12 Agents & agentic systems

### §12.1 Patterns (ReAct, Reflexion, Self-Refine, ToT)
- **L2:** ReAct = think + act + observe loop; Reflexion = verbal self-critique with memory; ToT = search over reasoning branches.
- **L4:** *"When does multi-step agent beat single LLM call?"* — uncertainty + tools + verifiable steps; for one-shot Q&A, simpler usually wins.

### §12.2 Tool use, function calling
- **L2:** schema design; descriptions matter; parallel calls; JSON-mode vs grammar; error handling.
- **L3:** BFCL eval for tool-call accuracy; tool name/description are prompt-engineered; tool-call hallucination on unseen tools.
- **L4:** *"Reduce hallucinated tool calls."* — tighter descriptions + few-shot examples + constrained decoding to valid schema + post-hoc validator.
- **Verticals:** AI-lab, FAANG (assistants), Consult (RPA).

### §12.3 Multi-agent (CrewAI, AutoGen, LangGraph)
- **L2:** state machine (LangGraph) vs conversation (AutoGen) vs role-based (CrewAI).
- **L4:** *"Is multi-agent overkill?"* — usually yes; single agent with tools wins on cost and reliability; multi-agent justified when roles have orthogonal context budgets.

### §12.4 Agent SDKs
- **L2:** Claude Agent SDK (subagents, hooks, MCP integration); OpenAI Agents SDK; Mastra; Vercel AI SDK.
- **L4:** *"Why Claude Agent SDK for production?"* — clean tool/hook abstractions, MCP-native, subagent isolation.

### §12.5 MCP (Model Context Protocol)
- **L2:** standard for tool servers; resources, tools, prompts primitives; stdio/SSE/streamable-HTTP transports.
- **L4:** *"Build an MCP server for your internal Postgres."* — tools = parameterized queries; resources = schema; auth at server boundary.

### §12.6 Computer-use, browser agents
- **L2:** screenshot + grid; Anthropic computer-use; OSWorld; WebArena benchmark.
- **L4:** *"Reliability issues with computer-use — fixes?"* — DOM-aware fallback, structured task decomposition, replay tests.

### §12.7 Agent memory
- **L2:** episodic / semantic / working; vector store as long-term; summarization for compression.
- **L4:** *"Design memory for a personal assistant that remembers user prefs over months."* — episodic event log + semantic fact extraction (LLM) + vector index + decay/relevance scoring + read-time retrieval.

### §12.8 Agent planning / decomposition
- **L2:** plan-and-execute, ReWOO, LLM Compiler.
- **L4:** *"When to plan upfront vs react?"* — long-horizon = upfront plan; dynamic environments = ReAct-style.

### §12.9 Agent eval, trace analysis
- **L2:** trajectory metrics; tool-use accuracy; step grading; LLM-judge on traces; failure-mode clustering.
- **L4:** *"Your agent passes unit tests but fails in prod — what eval did you miss?"* — distribution shift on user prompts; eval lacks long-tail; need trace-based prod eval + replay harness.
- **Verticals:** AI-lab, FAANG, Consult (your "traces" library territory).

### §12.10 Sandboxing
- **L2:** E2B/Modal/Daytona; Firecracker; gVisor; ephemeral vs persistent.
- **L4:** *"Code-exec agent — security model?"* — VM-level isolation, network egress restrictions, time/memory caps, audit logs.

### §12.11 Long-horizon reliability
- **L4:** *"Agent task fails 1% per step — 100-step task succeeds 37% — fix?"* — checkpoint + replay, smaller subgoals, verifier per step, recovery prompts.

---

### §13 Prompting & inference-time techniques

### §13.1 CoT family
- **L2:** zero-shot CoT ("Let's think step by step"); manual CoT; auto-CoT; self-consistency.
- **L4:** *"Self-consistency cost-benefit?"* — N samples + majority vote; cost = N×; gains on math/reasoning; not on retrieval Q&A.

### §13.2 BoN, rejection sampling
- **L2:** generate N, pick best by reward model or verifier; equivalent to first-step search.

### §13.3 Tree search (MCTS, A*)
- **L2:** PV-MCTS, AlphaZero-style for reasoning; pruning by PRM.
- **L4:** *"MCTS at inference for an LLM — practical?"* — only for deep reasoning where intermediate verifier exists; mostly research.

### §13.4 PRM + search
- **L2:** step-level reward as search heuristic; cite Math-Shepherd, "Let's Verify".

### §13.5 Proposer-reviewer
- **L2:** generate-critique-revise loop.
- **L4:** *"Cost vs quality tradeoff?"* — each loop ~2× cost; diminishing returns after 2–3 iterations.

### §13.6 Test-time compute scaling
- **L2:** Snell 2024 — compute-optimal allocation between CoT length and BoN; small + lots-of-CoT often beats big + greedy.
- **L4:** *"How does o1 work?"* — long CoT RL'd with verifiable rewards; inference budgets allocated to CoT length.

### §13.7 Prompt compression
- **L2:** LLMLingua iterative token dropping; AutoCompressor soft prompts.
- **L4:** *"Cut input tokens 80% — at what quality cost?"* — depends on info density; structured input compresses less.

---

### §14 Multimodal

### §14.1 VLMs
- **L2:** vision encoder + projector + LM; image-token budget; dynamic resolution; document VLMs.
- **L4:** *"VLM on tables — fix accuracy."* — high-res tiles + OCR augmentation + table-specific FT (Docling) + structured output.

### §14.2 Image gen
- **L2:** SD (U-Net + VAE + text-enc); FLUX (flow matching, DiT); CFG; LoRA; ControlNet; SDXL.
- **L4:** *"Flow matching vs diffusion — why win?"* — straighter trajectories, fewer steps, deterministic; cite rectified flow.

### §14.3 Video gen
- **L2:** spacetime patches (Sora); temporal consistency; Kling, Veo, Runway.

### §14.4 Speech
- **L2:** Whisper enc-dec; streaming ASR; TTS (Tortoise, XTTS, ElevenLabs); voice cloning.

### §14.5 Audio LLMs, music
- **L2:** AudioLM, MusicGen, Suno; codec models (EnCodec).

### §14.6 Embedding models
- **L2:** unified text-image-code; Matryoshka.

### §14.7 OCR + VLM
- **L2:** Docling, Nougat, GOT-OCR; pipeline vs VLM-direct.

---

### §15 Hardware

### §15.1 NVIDIA generations
- **L2:** Volta (V100, FP16) → Ampere (A100, BF16, 2:4) → Hopper (H100/H200, FP8, transformer engine) → Blackwell (B100/B200/GB200, FP4); HBM3/3e/4 capacity and BW.
- **L4:** *"H100 vs B200 for inference — when worth upgrading?"* — FP4 + bigger HBM + faster NVLink + double FLOPs; depends on workload memory-vs-compute mix.
- **Verticals:** AI-lab, FAANG, Fin (HFT-adjacent).

### §15.2 AMD MI300X
- **L2:** 192GB HBM (huge); ROCm stack; vLLM/SGLang support; closing software gap.
- **L4:** *"MI300X advantage?"* — single-GPU 70B+ inference without TP; cheaper $/GB; software still NVIDIA-favored.

### §15.3 TPUs
- **L2:** v4/v5e/v5p/Trillium/Ironwood; MXU; pod topology; XLA; JAX-native.
- **L4:** *"GPU → TPU port — pain points?"* — XLA compilation, SPMD partitioning, async dispatch; pjit/jit decorators.

### §15.4 Alt accelerators
- **L2:** Cerebras WSE (whole-wafer), Groq LPU (deterministic dataflow, ultra-low latency), SambaNova (reconfigurable), Tenstorrent (RISC-V), Etched Sohu (transformer ASIC).
- **L4:** *"Groq's latency claim — how?"* — on-chip SRAM, deterministic compilation, no batching overhead; tradeoff: limited model size.

### §15.5 Apple Silicon, NPUs
- **L2:** unified memory, MLX, Core ML; Snapdragon NPU.

### §15.6 Interconnects
- **L2:** NVLink (900 GB/s on H100), NVSwitch, InfiniBand HDR/NDR/XDR, RoCE v2, UALink; GPUDirect RDMA.
- **L3:** all-reduce algorithms (ring, tree, double-binary tree); when each wins.
- **L4:** *"Cross-rack training bottleneck?"* — IB or RoCE bandwidth; topology-aware NCCL; gradient compression.
- **CS:** networking, distributed systems.

### §15.7 Memory hierarchy
- **L2:** HBM bandwidth (~3 TB/s H100), SRAM in tensor cores, PCIe 5/6, CXL.
- **L4:** *"Roofline analysis on attention — where's it bound?"* — softmax is memory-bound (HBM), GEMM is compute-bound at large enough batch.

---

### §16 Safety, interpretability, security

### §16.1 Mech interp
- **L2:** circuits, induction heads, superposition, SAEs (sparse autoencoders), dictionary learning.
- **L3:** activation patching, causal scrubbing; transformer-circuits.pub series.
- **L4:** *"What's an SAE and what does it find?"* — overcomplete sparse decomp of activations into interpretable features; cite "Towards Monosemanticity".

### §16.2 Activation steering
- **L2:** add steering vector to residual stream to control behavior; refusal direction.

### §16.3 Jailbreaks, prompt injection
- **L2:** direct (GCG suffix), indirect (data exfiltration via injected docs); HarmBench.
- **L4:** *"Defense in depth for prompt injection?"* — content separation in prompt, input filter, output filter, tool-call review, least-privilege tools.
- **Verticals:** AI-lab, Consult (enterprise security), Fin.

### §16.4 Red teaming
- **L2:** automated (PAIR, TAP, rainbow); manual; HarmBench, AdvBench.

### §16.5 Watermarking, provenance
- **L2:** logits-based watermarks (Kirchenbauer); C2PA cryptographic provenance.

### §16.6 Membership inference, training data extraction
- **L2:** canary insertion; extraction attacks; differential privacy as defense.
- **L4:** *"How would you prove your data isn't in a model?"* — query attack, log-prob ratio, canary; difficult without ground truth.

### §16.7 Differential privacy
- **L2:** (ε, δ); DP-SGD; privacy accounting (RDP, GDP).
- **L4:** *"DP-SGD impact on accuracy?"* — major at small ε; clipping + noise hurt convergence; usually 1–10 utility drop at ε=8.
- **Verticals:** Fin, Consult.

### §16.8 Alignment research
- **L2:** scalable oversight, weak-to-strong, debate, ELK.

---

### §17 Performance profiling

### §17.1 Nsight Systems / Compute
- **L2:** NS for timeline; NC for per-kernel occupancy, memory throughput, SM utilization.
- **L4:** *"Walk through profiling a slow inference path."* — Nsight Systems timeline → spot gap → drill into kernel with NC → check memory/compute roofline → fix.
- **CS:** GPU programming.

### §17.2 PyTorch Profiler, HTA
- **L2:** record_function, Chrome trace, HTA for distributed insight.

### §17.3 Roofline analysis
- **L2:** AI = FLOPs / bytes; peak FLOPs vs peak BW lines; placement of ops.
- **L4:** *"GEMM with K=4096, M=N=128 — bound by what?"* — small batch → memory-bound on H100; large batch → compute.

### §17.4 MFU / HFU
- **L2:** model FLOPs / hardware peak; 30–50% is good for training; lower for inference.

### §17.5 Memory profiling
- **L2:** torch.cuda.memory_snapshot, memory_viz, py-spy.

### §17.6 Bottleneck analysis
- **L2:** compute-bound vs memory-bound vs comm-bound symptoms.

### §17.7 Microbenchmarks
- **L2:** FA-bench, vLLM bench, MLPerf inference.

### §17.8 Latency decomposition
- **L2:** TTFT, ITL, throughput, queueing.
- **L4:** *"User reports 'feels slow' — decompose."* — client → LB → queueing → prefill → first token → decode rate → streaming; instrument each.

---

## 4. Interview-format playbook by vertical

### AI labs (Anthropic, OpenAI, Google DeepMind, xAI, Mistral, DeepSeek)
- **Coding (45–60 min):** implement attention from scratch; KV cache; greedy/beam decode; data loader; sometimes Triton kernel.
- **ML breadth (45 min):** rapid-fire across §§1–7; transformer derivations; scaling laws; RLHF/DPO/GRPO comparison.
- **ML depth (60+ min):** one paper deep-dive; whiteboard derive; failure modes; how would you improve it.
- **Systems (60 min):** design inference for X model at Y QPS; KV cache, batching, parallelism choice.
- **Research-y problem (take-home or live):** small open-ended problem ("how would you improve speculative decoding?").
- **Behavioral / mission fit:** alignment, safety beliefs, motivation.
- **What to overprepare:** §3, §7, §8, §9, §17.

### FAANG SaaS / consumer (Meta, Google products, Apple, Microsoft, Amazon, Netflix)
- **Coding (2–3 LeetCode-style rounds):** algorithms + ML algorithm implementation.
- **ML system design (60 min):** "design TikTok recs," "design Gmail Smart Compose," "design YouTube search."
- **ML breadth/depth:** broad, with focus on production ML, A/B testing, drift, monitoring.
- **Behavioral:** Amazon LP / Meta values style.
- **What to overprepare:** §10 (all of it), §11, §12, A/B testing, recsys foundations, mock system designs.

### Data engineering / data platforms (Databricks, Snowflake, Confluent, dbt Labs, Fivetran)
- **Coding:** SQL-heavy; distributed data structures; streaming algorithms.
- **System design:** "design a lakehouse table format," "design vector search at petabyte scale," "design real-time feature store."
- **ML knowledge:** working ML breadth, vector DB internals, embedding pipelines, feature engineering at scale.
- **What to overprepare:** §10d, §11.11, distributed systems, columnar formats (Parquet, Arrow), query optimization, CDC.

### Finance / quant (Two Sigma, Jane Street, Citadel, Renaissance, JPMorgan AI)
- **Coding:** algorithms + stats / probability problems live.
- **ML/stat:** Bayesian, causal, time-series, RL; rigor on uncertainty.
- **System design:** low-latency systems; risk; compliance.
- **What to overprepare:** §1.6, §1.2, §1.7, §16 (esp. DP, model risk), §10h, time-series modeling, CLT/bootstrap, A/B at scale.

### Consulting (McKinsey QuantumBlack, BCG GAMMA, Accenture AI, Deloitte AI&D, big-4)
- **Case interview:** structured problem framing, MECE; AI/ML case studies (lender churn, predictive maintenance, GenAI for a CPG).
- **ML breadth:** must speak across all topics at L2 with clarity; less L3/L4 depth than AI labs.
- **Communication:** explain Transformer to a CFO in 3 minutes.
- **What to overprepare:** §4 (model selection criteria), §10 (lifecycle, ROI math), §11–12 (RAG/agents for enterprise), §10h compliance, soft skills (storytelling, frameworks).

---

## 5. Cross-cutting CS connections (don't miss in interviews)

| Topic | Crosses into |
|---|---|
| Attention KV cache | OS virtual memory (paging), DB buffer pools |
| Speculative decoding | Branch prediction, speculative execution in CPUs |
| Continuous batching | OS scheduling, queueing theory |
| Quantization | Numerical methods, fixed-point arithmetic |
| Embeddings + ANN | Hashing (LSH), spatial data structures, IR |
| RAG | Search systems, IR (BM25), DBs |
| Tokenizers | Compression, tries, suffix automata |
| MoE routing | Load balancing, consistent hashing |
| Distributed training | MPI, collective comm, fault tolerance |
| Diffusion sampling | SDEs, numerical integration |
| RL | DP (Bellman), control theory, game theory |
| Model serving | Web infra (gateways, rate limiting, autoscaling) |
| Vector DBs | DB internals (LSM, indexes), distributed storage |
| Mechanistic interp | Circuits (digital design analog), causal inference |
| Agent reliability | Distributed systems (idempotency, retries, sagas) |

---

## 6. The minimum-viable canon (read these 20 things)

Pick 20 anchor reads — the rest you can skim. Suggested set:

1. "Attention is All You Need" (Vaswani 2017)
2. "Language Models are Few-Shot Learners" (GPT-3, Brown 2020)
3. "Chinchilla scaling laws" (Hoffmann 2022)
4. "InstructGPT" (Ouyang 2022)
5. "DPO" (Rafailov 2023)
6. "DeepSeek-V3 / R1" technical reports (2024–2025)
7. "FlashAttention v1" (Dao 2022)
8. "PagedAttention / vLLM" (Kwon 2023)
9. "LoRA" (Hu 2021)
10. "GPTQ" (Frantar 2022) and "AWQ" (Lin 2023)
11. "Speculative Decoding" (Leviathan 2022; Chen 2023)
12. "EAGLE" (Li 2024)
13. "RoPE" (Su 2021)
14. "Mixture of Experts (Switch Transformer)" (Fedus 2021)
15. "Mamba" (Gu & Dao 2023)
16. "RAG" (Lewis 2020) + "Contextual Retrieval" (Anthropic 2024)
17. "ReAct" (Yao 2022)
18. "Constitutional AI" (Bai 2022)
19. "Mech interp: A Mathematical Framework for Transformer Circuits" (Anthropic 2021)
20. "Let's Verify Step by Step" (Lightman 2023)

Plus one engineering write-up at industrial depth: the Mooncake or DistServe paper, depending on your inference-systems angle.

---

## 7. Self-assessment checklist (use weekly)

- [ ] Can I derive softmax attention complexity in 60s?
- [ ] Can I derive DPO from KL-regularized RLHF?
- [ ] Can I compute KV cache size for any model spec given GQA params?
- [ ] Can I size a training run from Chinchilla + inference-aware adjustments?
- [ ] Can I design a multi-tenant LLM serving system end-to-end?
- [ ] Can I name 5 sampler variants and when each helps?
- [ ] Can I list the 4 models alive during PPO and their memory cost?
- [ ] Can I explain why Mamba struggles with copying?
- [ ] Can I diagnose a "model feels slow" complaint top-to-bottom?
- [ ] Can I explain prompt-injection defense-in-depth?
- [ ] Can I name 3 vector DB index trade-offs at billion scale?
- [ ] Can I design an agent eval that catches silent failures?

If any of these is "no", that's your next focus.
ritten over python Flask/FastAPI