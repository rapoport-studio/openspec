# Training the Orchestra to Author OpenSpec Specs — May 2026

> Source-grounded research pass on the 8 gaps the internal digest flagged (prompt-cache layering, RAG-on-small-corpora, self-improvement loops, eval harnesses, code+spec embeddings, human factors, AI-author risks). Companion to the existing `openspec/_methodology/research/` library — does **not** repeat the 4-layer cache strategy, AI SDK v6 / MCP / Skills choice, the seven OpenSpec principles, the trust-gradient red lines, or the Mem0/Letta/Cognee triage that's already on file.

## TL;DR

The orchestra can become a competent OpenSpec author without breakthrough research — every primitive needed (linting, RAG, caching, eval, embedding, sycophancy mitigation) has matured in 2025. The constraint is **assembly discipline**, not capability. Three findings dominate:

1. **Vale + an LLM-as-judge rubric is the cheapest, biggest-ROI Layer-1 win.** Vale handles deterministic prose rules (terminology, readability, sentence shape) in <2 s on 1.5 k Markdown files (Datadog benchmark). G-Eval / SycEval-style LLM judges plug the gap Vale can't see (completeness, internal consistency). Both produce a per-spec score that becomes the gradient signal for everything else in this document.
2. **RAG is overkill at our scale.** With a corpus of ~100–500 spec files, the whole `openspec/` bundle fits inside Claude / Gemini long-context windows and a hybrid BM25 + dense retriever is needed only when total context exceeds ~200 K tokens. The 2026 RAG-vs-LC literature converges on this threshold.
3. **GEPA-style reflective prompt evolution beats hand-tuning by +10 pp on average and is the right "self-improvement" lever** — but the Reflexion / self-critique literature is unanimous that the marginal return falls off a cliff after 2–3 iterations. Build the optimization loop, but cap the loop.

The biggest open empirical question for our domain is **whether LLM-as-judge for prose-style specs (vs. OpenAPI / formal specs) survives the order-bias and preference-leakage problems documented in the SycEval and RESpecBench 2025 papers.** No public benchmark yet covers "is this Markdown specification well-written" the way RESpecBench covers "is this generated formal spec semantically equivalent." This is a gap Rapoport could fill cheaply.

---

## Layer 1: Spec quality scoring & linting

### What's available off the shelf

| Tool | What it measures | Determinism | Useful for OpenSpec |
|---|---|---|---|
| **Vale** ([vale.sh](https://vale.sh/docs)) | Terminology, capitalization, repetition, occurrence, conditional rules, readability metrics, sentence-shape regex — all via plain-YAML rules | 100% (rule-based) | Yes — Datadog, Grafana, Spectro, Contentsquare use it in CI; runs 1.5 k MD files <2 s |
| **proselint** ([github.com/amperser/proselint](https://github.com/amperser/proselint)) | "Naive" English style — clichés, illogic, dates, jargon | 100% | Marginal — fewer rules than Vale, less customizable |
| **alex** ([alexjs.com](https://github.com/get-alex/alex)) | Inclusive-language linter (gender, race, religion-insensitive phrasing) | 100% | Yes for tone hygiene, but small surface |
| **retext** ([retextjs.github.io](https://retextjs.github.io)) | Plugin-based — readability, simplification, indefinite articles, equality | 100% | Useful as a library if we want to write our own checks |
| **write-good** ([github.com/btford/write-good](https://github.com/btford/write-good)) | Passive voice, redundant words, "weasel" words | 100% | Marginal — covered by Vale |
| **textstat / py-readability-metrics** ([github.com/cdimascio/py-readability-metrics](https://github.com/cdimascio/py-readability-metrics)) | Flesch-Kincaid, Gunning Fog, ARI, Dale-Chall, SMOG | 100% | Yes — Vale already wraps several of these |

### Specification-quality research (IEEE 830 era to 2025)

The requirements-engineering community has been at this for 25 years and the academic toolkit transfers directly:

- **QuARS** (Quality Analyzer for Requirements Specifications, [ITTI-CNR](https://ceur-ws.org/Vol-2376/NLP4RE19_paper07.pdf)) — rule-based NLP for ambiguity detection. Limited dictionaries, no reported precision/recall against a held-out benchmark.
- **Smella** — POS-tagging + lemmatization to detect 4 of the 8 IEEE-830 "smells." Outperformed by newer span-tagging approaches; a 2025 SpanBERT + NER study ([Wiley](https://onlinelibrary.wiley.com/doi/10.1002/smr.70041)) showed semi-automated SpanBERT detection beats Smella on ~1000 industrial requirements.
- **Element Quality Indicator (EQI)** — assigns per-element scores against IEEE-830 properties (completeness, correctness, preciseness, consistency).
- **Rule-based NLP vs. ChatGPT in ambiguity detection** ([CEUR Vol-3378](https://ceur-ws.org/Vol-3378/NLP4RE-paper1.pdf)) — preliminary 2023 study showed LLMs catch ambiguity that pure rule-based misses, but with lower precision than QuARS on rule-coverable cases. Hybrid wins.

### LLM-as-judge state of the art

- **RESpecBench** ([OpenReview](https://openreview.net/forum?id=eFwJZIN9eI)) — first rigorous benchmark for LLM-as-judge on **specification generation** (GSM-Symbolic+, SQL, FOL, RegEx, Rocq). Covers semantic-equivalence judgment, **not prose-style specs**. Confirms LLM-judge has become the dominant assessment methodology but reliability is poorly studied for the spec-authoring use case.
- **SycEval** ([arxiv.org/abs/2502.08177](https://arxiv.org/abs/2502.08177)) — judge models exhibit sycophantic behavior in 58.19% of cases (Gemini 62.47%, GPT-4o 56.71%, Claude in between). 14.66% **regressive** sycophancy (agrees with wrong human). Direct threat to "use Opus to grade Sonnet's specs."
- **Preference Leakage** (ICLR 2026) — contamination problem where the judge model has overlap with the generator's training data inflates scores. Mitigation: use a judge from a different model family than the author.
- **LLM-as-Judge survey** ([arxiv.org/html/2412.05579v2](https://arxiv.org/html/2412.05579v2)) — order-swap bias >10 pp accuracy shift in pairwise judging.
- **2026 industry guides** ([Label Your Data](https://labelyourdata.com/articles/llm-as-a-judge), [Evidently](https://www.evidentlyai.com/llm-guide/llm-as-a-judge), [Monte Carlo](https://www.montecarlodata.com/blog-llm-as-judge/)) — converge on three best practices: (1) explicit rubric in the prompt, not a single score; (2) chain-of-thought reasoning before score (+10–15% reliability); (3) judge from a different model family than the generator.

### Gaps for Rapoport

1. **No public benchmark for "is this Markdown spec well-written"** for prose-style specs the way RESpecBench exists for formal specs. Building 30–50 hand-graded OpenSpec examples would be the cheapest "scientifically novel" artifact in this whole roadmap.
2. **Vale's rule surface is per-document, not cross-document.** Detecting "this spec contradicts platform.md § X" needs LLM-judge.
3. **Readability metrics (Flesch-Kincaid) target grades 9–13 for technical material** ([Docsie 2025 guide](https://www.docsie.io/blog/glossary/flesch-kincaid/)) — but OpenSpec specs are read by both humans and the orchestra, so the target may need to be domain-tuned.

---

## Layer 2: RAG for spec corpora at small scale

### Long-context vs. RAG decision point (2026 consensus)

The 2026 RAG-vs-LC literature has converged on token thresholds:

- **<200 K tokens**: skip RAG entirely. Use any 200-K-class model, paste the whole bundle. ([Tianpan 2026](https://tianpan.co/blog/2026-04-09-long-context-vs-rag-production-decision-framework), [Open-techstack](https://open-techstack.com/blog/rag-vs-long-context-2026/))
- **200 K – 1 M**: long-context with prefix caching. Latency 30–60× higher than RAG, cost 1,250× per query (Sitepoint, citing Databricks long-context RAG benchmark). Only worth it when retrieval recall hurts you.
- **>1 M**: hybrid — RAG to filter, long-context to reason.

**Lost-in-the-middle remains real.** Liu et al. ([TACL 2024](https://aclanthology.org/2024.tacl-1.9/), [arxiv 2307.03172](https://arxiv.org/abs/2307.03172)) measured a 30+ pp accuracy drop when the relevant doc moves from position 1 → 10 in a 20-doc context. U-shaped attention curve confirmed across GPT-3.5, GPT-4, Claude 1.3, MPT-30B, LLaMA-2. Even with a 1 M-token window, **placement of the relevant spec matters more than retrieval mechanics** at our scale.

### `openspec/` size today

The OpenSpec bundle module is gzip-capped at 6 MB (RAP-697). Raw markdown across `current/` + `archive/` + active `changes/` is on the order of a few hundred KB to low MB — comfortably in the <200 K-token bucket for any modern model. **Conclusion: RAG is not the bottleneck. Order of injection is.**

### When RAG becomes necessary

If/when the corpus grows past ~200 K tokens (active changes inflate fast) — or if cost optimization makes loading the full bundle wasteful per turn — the modern stack:

| Layer | Best 2025–26 choice for small corpus |
|---|---|
| Embedding | **BGE-M3** ([HuggingFace](https://huggingface.co/BAAI/bge-m3)) — multilingual, 8192-token context, unifies dense + sparse + ColBERT in one encoder. Or **Voyage-3-large / voyage-code-3** ([Voyage blog](https://blog.voyageai.com/2024/12/04/voyage-code-3/)) — Voyage-code-3 beats OpenAI text-embedding-3-large by 13.80% on average across 32 code retrieval datasets, at 1/3 the storage cost. |
| Vector store | **pgvector 0.9** ([CallSphere 2026 benchmark](https://callsphere.ai/blog/vector-database-benchmarks-2026-pgvector-qdrant-weaviate-milvus-lancedb)) for Supabase-native (we already have Postgres). **LanceDB** ([Zilliz comparison](https://zilliz.com/comparison/pgvector-vs-lancedb)) for embedded; **sqlite-vec** absent from 2025 benchmarks, indicating immaturity. |
| Sparse | **BM25** via `rank_bm25` or built-in Postgres `ts_vector` — recall+10ish pp on keyword-heavy queries (DEV/Qvfagundes hybrid retrieval write-up, [Ranjan Kumar full-stack hybrid](https://ranjankumar.in/building-a-full-stack-hybrid-search-system-bm25-vectors-cross-encoders-with-docker)) |
| Fusion | **Reciprocal Rank Fusion (k≈60)** — adds ~6 ms p50, classic recipe ([Glaforge 2026](https://glaforge.dev/posts/2026/02/10/advanced-rag-understanding-reciprocal-rank-fusion-in-hybrid-search/)). |
| Reranker | **Optional** at <1 k docs. Cross-encoder reranking pays off only when the retriever returns ≥100 candidates and you compress to ~10 ([Progress blog](https://www.progress.com/blogs/master-advanced-search-ranking-fusion-and-reranking-explained), [Vizuara](https://vizuara.substack.com/p/a-primer-on-re-ranking-for-retrieval)). For <500 docs total, RRF on top-20 → LLM is usually enough; reranker is gold-plating. |

### Chunking for Markdown specs

- **Structural chunking by heading** (## / ###) is the dominant pattern for spec-style markdown — preserves "what this section is about" in metadata.
- **Late chunking** ([Jina, arxiv 2409.04701](https://arxiv.org/pdf/2409.04701)) — embed the whole doc first, then derive chunk vectors from token-level reps. Avoids losing cross-chunk context (e.g., "the term defined in §1 used in §3"). Available via `jina-embeddings-v3` `late_chunking` API. Worth piloting for any spec that has cross-references — which is most OpenSpec specs.
- **Hybrid recall lift**: BM25 + dense + RRF lifts recall@10 from 65–78% → 91% on technical text in published benchmarks ([Supermemory hybrid guide](https://blog.supermemory.ai/hybrid-search-guide/)).

### Where this layer matters for OpenSpec

Probably not at all for the next 12 months. The "RAG question" reduces to "which 200 K tokens do I show Claude this turn." That's a **bundling and ordering problem**, not a retrieval problem. The right Layer-2 investment is the deterministic bundle builder we already have, plus a "what was changed since last bundle" diff so the cache hit rate stays high.

---

## Layer 3: Caching beyond the 4-layer pattern

The internal digest already has the 4-layer plan (L1 system 5 min, L2 tools/skills 5 min, L3 bundle/repo-map 1 h, L4 dynamic). Additions worth knowing:

### Anthropic 1-hour TTL pricing economics

- 1-hour cache **write** cost: 2× base input ($6/M vs $3/M for Sonnet) ([prompt-caching docs](https://platform.claude.com/docs/en/build-with-claude/prompt-caching), [Spring AI blog 2025-10-27](https://spring.io/blog/2025/10/27/spring-ai-anthropic-prompt-caching-blog/)).
- 5-minute cache **write** cost: 1.25× base.
- **Read** cost (both TTLs): 0.1× base.
- Break-even: if a 1-h cached prefix is re-read **>8 times within the hour**, the 1-h TTL beats the 5-min refresh model. The architect role + bundle (read once per build) fits this, the orchestrator's per-task system prompt (read many times within minutes) does not.

### Stable-prefix design pattern

The rule across vendors ([Bedrock](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-caching.html), [F5 KV-cache-aware engineering](https://ankitbko.github.io/blog/2025/08/prompt-engineering-kv-cache/), [Anthropic](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)):

> **Stable at front, volatile at back.** Anything that mutates between requests — request IDs, timestamps, the current user query — goes at the end.

LMCache's reverse-engineering of Claude Code measured **92% prompt reuse across phases** ([LMCache 2025-12](https://blog.lmcache.ai/en/2025/12/23/context-engineering-reuse-pattern-under-the-hood-of-claude-code/)). That's the bar.

For OpenSpec, the implication: **never inject the Linear issue body or branch name above the openspec/ bundle.** Today some prompts probably do — worth auditing.

### Cache invalidation strategies for spec corpora

The cache hierarchy is **tools → system → messages** ([Anthropic docs](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)) — any change to a level invalidates that level and below.

Two patterns from the broader cache-invalidation literature ([Tianpan 2026 cache-invalidation](https://tianpan.co/blog/2026-04-20-cache-invalidation-ai-semantic-rag), [MIT CSAIL Merkle memo-453](https://csg.csail.mit.edu/pubs/memos/Memo-453/memo-453.pdf)) transfer cleanly:

1. **Content-hash the bundle.** Include `bundle_sha256` in the cache key. Auto-invalidate on any spec edit. We already have this primitive via the bundle build script.
2. **Merkle-tree the corpus.** Hash each spec file, build a tree, cache the tree root. Only invalidate the subtrees a given prompt actually used. Overkill at <500 files — flag for the day the bundle module grows past 6 MB and we move it to KV/R2 (memory note already on file).

### Semantic caching (cache by query similarity)

[**GPTCache**](https://github.com/zilliztech/GPTCache) is the open-source reference. Embeds queries, looks up the K nearest in a vector store (FAISS / Milvus / Zilliz), returns the cached response if the similarity passes threshold. Three metrics: hit ratio, latency, recall.

**Useful for OpenSpec? Probably not yet.** Two reasons:
- Spec-authoring queries are highly heterogeneous (different Linear issues, different files touched). Cache hit rate will be low.
- The risk is *wrong* hits — semantic match on the surface, divergent intent below. For spec authoring this is worse than a miss.

Better fit: **spec Q&A surfaces** (e.g., "what does platform.md say about Y?") where queries cluster. Worth pilot if/when the platform surfaces an internal Q&A endpoint.

---

## Layer 4: Agent self-improvement loops

### GEPA / DSPy / BAML — the 2025–26 winner stack

**GEPA** (Genetic-Pareto reflective prompt evolution, [arxiv 2507.19457](https://arxiv.org/abs/2507.19457), ICLR 2026 oral) is the headline result of 2025:

- Beats GRPO (RL) by **6% on average, up to 20%, with up to 35× fewer rollouts**.
- Beats MIPROv2 (previous SOTA prompt optimizer) by **>10%** — e.g., +12 pp on AIME-2025.
- 93% on MATH benchmark vs 67% baseline with DSPy ChainOfThought.

Production case study ([Kevin Madura's writeup](https://kmad.ai/DSPy-Optimization), Dec 2025): DSPy + GEPA + BAML adapter achieved **+20 percentage points** on a structured-extraction task. ~1,200 rollouts, ~$2–3 in API costs. Biggest gains came on prompt-sensitive categories.

**BAML schema format** ([Vicente Reig 2025-09](https://vicentereig.github.io/dspy.rb/blog/articles/baml-schema-format/), [Prashanth Rao benchmark](https://github.com/prrao87/structured-outputs)) — 84.4% aggregate token reduction (632 → 99 tokens) across structured-output tests at 100% quality parity. Lean schema, rich signature. Adopted as the schema format for DSPy adapters.

**Action**: GEPA + BAML is the right "compile-the-prompts" loop for the orchestra's spec-authoring roles. Modest cost, large quality lift, plays cleanly with the existing 4-layer cache because optimized prompts can sit in L2 (skill layer).

### Reflexion / self-critique — empirical limits

**Reflexion** ([Shinn et al. 2023, NeurIPS](https://www.semanticscholar.org/paper/Reflexion:-an-autonomous-agent-with-dynamic-memory-Shinn-Labash/46299fee72ca833337b3882ae1d8316f44b32b3c)) introduced verbal RL via self-reflection. Followups settled the diminishing-returns question:

- **MAR (Multi-Agent Reflexion, [arxiv 2512.20845](https://arxiv.org/html/2512.20845))** — "sharply diminishing returns beyond certain iteration thresholds" — caps debates at 2 rounds.
- **EMNLP 2025 "Probabilistic Inference Scaling Theory for LLM Self-Correction"** — formalizes the ceiling: a model cannot fix errors it cannot recognize.
- **Long-Horizon Execution paper ([arxiv 2509.09677](https://arxiv.org/html/2509.09677v1))** — "self-conditioning effect": past errors in the model's own output *increase* the chance of future mistakes. Long Reflexion chains poison themselves.

**The 2–3-iteration cap is now empirically established.** Reflexion as a one-shot critique pass before submission: useful. Reflexion as an open-ended loop: don't.

### Chain-of-Verification (CoVe)

[CoVe (arxiv 2309.11495, ACL 2024)](https://aclanthology.org/2024.findings-acl.212/) — draft → plan verification questions → answer them in isolation → revise. Reported gains: list-QA precision 0.17 → 0.36 on Llama 65B; FactScore on long-form bio 55.9 → 71.4. Plays well with spec authoring because OpenSpec specs have plenty of verifiable sub-claims ("does this spec cite the right capability file?").

### AutoResearch — the speculative end

[Andrej Karpathy's nanochat autoresearch](https://www.latent.space/p/ainews-autoresearch-sparks-of-recursive) (early 2026) ran ~700 autonomous changes and found ~20 stacking improvements (~11% wall-clock gain on "Time to GPT-2"). [Bilevel Autoresearch](https://arxiv.org/html/2603.23420), [AutoResearch-RL](https://arxiv.org/abs/2603.07300), [OmniMem](https://arxiv.org/html/2604.01007v1), [ARIS](https://arxiv.org/html/2605.03042v1) — all 2026 — extend the loop to multi-modal memory, NAS, and adversarial multi-agent collaboration.

For spec authoring this is **too speculative for a 1–3-person studio**. Worth knowing the literature exists; not worth building before the GEPA/BAML loop is in production and producing measurable wins.

### Constitutional AI for spec authoring

Anthropic's [Constitutional AI](https://www.anthropic.com/research/constitutional-ai-harmlessness-from-ai-feedback) framework — train against a list of principles via self-critique. The [2025 new constitution](https://time.com/7354738/claude-constitution-ai-alignment/) plays a central role in Claude's training. **For our case, the framework transfers as a pattern, not a training technique:** the seven OpenSpec principles already in `_methodology/` are a "constitution" the authoring role can self-check against at draft time. This is a **prompt-level adoption**, not a model-training one.

---

## Layer 5: Eval harnesses for spec authoring

### Platform comparison (1–3-person studio context)

| Platform | Free tier | Best for | Notes |
|---|---|---|---|
| **Braintrust** ([articles](https://www.braintrust.dev/articles/langfuse-alternatives-2026)) | 1 M trace spans + 10 K eval runs, unlimited users | Teams that wire eval into CI/CD; "quality management system for AI products" | Most generous free tier of the modern crop |
| **Langfuse** | Open source MIT, self-host or cloud; acquired by ClickHouse Jan 2026 | Open-source-first teams that want full data control | Tracing + prompt mgmt + eval; mature SDK |
| **Phoenix (Arize OSS)** | 25 K trace spans / 1 user | OpenTelemetry-native stacks, Postgres/K8s ops capacity | Full-OSS choice |
| **Helicone** | Generous; proxy model | Simplest setup (~15 min) if OpenAI is your only provider | Less suited to multi-model orchestras |
| **LangSmith** | Tied to LangChain | Heavy LangChain users | Less relevant here |

**Recommendation**: Braintrust for the eval/observability seam — best free tier, CI-native, integrates Trace-to-Test. Langfuse as backup if data residency or OSS purism becomes a constraint.

### Eval frameworks (the layer below the platform)

| Framework | Best for |
|---|---|
| **DeepEval** ([github](https://github.com/confident-ai/deepeval)) | 14+ prebuilt metrics, pytest CI/CD integration, G-Eval implementation. Strongest dev-time tool for chatbot/agent/RAG |
| **Ragas** ([deepeval doc](https://deepeval.com/docs/metrics-ragas)) | Reference-free RAG metrics (faithfulness, relevance, context quality) + synthetic data gen. Narrower but opinionated |
| **promptfoo** ([genai.qa comparison](https://genai.qa/blog/promptfoo-vs-deepeval-vs-ragas/)) | YAML-driven cross-model comparison & red-teaming. "Which model and how do I harden my prompts" |

The 2025–26 convergence ([Inference.net guide](https://inference.net/content/llm-evaluation-tools-comparison/)): experienced teams pair **one lightweight framework for CI gates** with **one platform for human annotation + dashboards.** For us, the natural pairing is **DeepEval (CI) + Braintrust (platform)** — both work in TypeScript.

### Golden dataset size

Consensus across [Microsoft promptflow](https://github.com/microsoft/promptflow-resource-hub/blob/main/sample_gallery/golden_dataset/copilot-golden-dataset-creation-guidance.md), [Maxim](https://www.getmaxim.ai/articles/building-a-golden-dataset-for-ai-evaluation-a-step-by-step-guide/), [Musubi](https://www.musubilabs.ai/post/how-to-build-golden-datasets-for-content-moderation), [Statsig](https://www.statsig.com/perspectives/golden-datasets-evaluation-standards), [Dextralabs production-RAG 2025](https://dextralabs.com/blog/production-rag-in-2025-evaluation-cicd-observability/), [Codersarts](https://www.codersarts.com/post/building-a-golden-dataset-and-evaluating-retrieval-quality):

- **MVP**: 30–50 examples for one policy/capability area
- **Statistical relevance**: ~100 Q&A pairs
- **Coverage**: 150–200 for a mature product

For spec authoring specifically — **start with one capability spec** (e.g., `platform.md` or `forge.md`), construct 30–50 graded pairs (prompt: "write a spec section for X"; gold: hand-written reference + rubric scores), iterate. This becomes both the eval harness *and* the training corpus for the GEPA loop. Same artifact, two uses.

### Regression cadence

Industry default (Braintrust, Statsig, Maxim guides): run the golden set in CI on every spec-authoring prompt change, plus a weekly full-fleet run with human review of 5–10 samples. Cost of a 50-example DeepEval pass at Sonnet/Haiku rates is roughly $0.10–$0.50.

---

## Layer 6: Code + spec joint embedding

### Where the field is (2025–26)

- **Voyage-code-3** ([Voyage blog 2024-12](https://blog.voyageai.com/2024/12/04/voyage-code-3/), [MongoDB 2025](https://www.mongodb.com/company/blog/voyage-code-3-more-accurate-code-retrieval-lower-dimensional-quantized-embeddings)) — beats OpenAI text-embedding-3-large by 13.80% on 32 code retrieval datasets; supports Matryoshka dims (1024 / 256) + int8/binary quantization for storage savings.
- **OpenAI text-embedding-3-small** — 95% of Voyage code-3's performance at $0.02/M tokens (vs $0.06/M). Strong default for cost-sensitive code retrieval.
- **CodeBERT / GraphCodeBERT / UniXcoder** are largely obsoleted by modern specialized embeddings (Modal blog benchmark: CodeBERT MRR 0.117 vs Voyage-code-3 0.973).
- **BGE-M3** unifies dense + sparse + ColBERT in one encoder, multilingual, 8192-token context — best single-model choice if you want one embedding for both code and prose. Late-interaction (ColBERT-style) gives sub-token relevance scoring suitable for technical text.

### Cross-modal retrieval ("show me spec for this code" / vice versa)

Active research area in 2025 ACL/EMNLP:

- [Joint code-text embeddings via contrastive loss](https://arxiv.org/html/2405.04126v1) — aligns matching pairs in a shared space using parameter-efficient fine-tuning.
- [UniRAG](https://aclanthology.org/2025.acl-long.693.pdf) — unified query understanding for code+text retrieval.
- [Large Reasoning Embedding Models](https://arxiv.org/html/2510.14321v1) — pushes embeddings beyond surface semantics toward "encoded inferential and procedural knowledge."

### What working code-AI tools actually do

- **Cursor** — AI-native VS Code fork, **dynamic context loading** (no full repo pre-index). Speed wins over completeness.
- **Sourcegraph Cody** — used to be embeddings-first; **switched away from embeddings to Sourcegraph Search** for Cody Enterprise. Hybrid: code-intel + search API + 1 M-token Gemini Flash context. Discontinued Free/Pro in July 2025.
- **Continue.dev** — embeddings-based RAG, configurable local or cloud embedding models, two-stage retrieval with re-ranking, MCP for external sources.
- **GitHub Copilot Workspace / Augment Code** — augment.code's positioning is "embeddings at monorepo scale" — claims advantage over Cursor's dynamic loading for large repos.

Pattern: **at small scale (single-repo, <500 files), dynamic context loading beats embeddings; at enterprise/monorepo scale, embeddings + search-API hybrid wins.** Rapoport is firmly in "dynamic context loading" territory for the next 12+ months.

### For OpenSpec specifically

The joint corpus is small enough that **a single embedding space — BGE-M3 or Voyage-code-3 — covers both directions ("which spec describes this code", "which code implements this spec").** Don't build two indexes. Don't fine-tune anything until evals show a measurable gap.

---

## Layer 7: Human factors — training people on OpenSpec

### Diataxis — empirical status

[Diataxis](https://diataxis.fr/) is widely adopted (Python core docs, GitLab, Cloudflare, Datadog), but **rigorous empirical validation is thin.** Daniele Procida grounds the framework in user needs and information-architecture theory, not controlled experiments. The closest thing to empirical backing is **adoption + qualitative reports** (Bernard 2024, Python.org discussion thread) — useful, not conclusive.

**Cognitive Load Theory** (Sweller, 1988, refreshed 2024 in [Sweller 2024 chapter](https://www.sciencedirect.com/science/article/pii/S1041608024000165)) is the implicit underpinning — Diataxis's quadrant separation maps to reducing extraneous load by keeping tutorial/reference/explanation/how-to mental models in separate documents. **No published study yet measures Diataxis adoption against documentation comprehension outcomes.**

For OpenSpec the practical takeaway is: **Diataxis's quadrant model is a stylistic guide, not a measured intervention.** Adopt the structure for its low-cost benefits; don't claim measurable productivity gains without measuring.

### ADR adoption studies

The empirical literature on Architecture Decision Records is small but growing:

- **["One Size Fits All? An Empirical Comparison of ADR Templates"](https://arxiv.org/abs/2604.27333)** — compares Tyree/Akerman, Nygard, arc42, Y-statements, MADR. Findings: **Nygard supports concise/objective documentation, MADR facilitates structural detail**. Decision guide to pick template by project constraint.
- **[Architecture Decision Records in Practice: An Action Research Study (ECSA 2024)](https://rebekkaa.github.io/files/2024_ECSA.pdf)** — describes the cultural + tooling + knowledge-mgmt challenges of introducing ADRs in real companies.
- **[Can LLMs Generate Architectural Design Decisions?](https://arxiv.org/html/2403.01709)** — exploratory empirical study on LLM-generated ADRs; mixed quality.

Common adoption failure modes documented across these studies: time pressure, inconsistent uptake, "we'll write it later" (never). The lesson for OpenSpec: **the template is the cheapest variable; consistent enforcement (CI, PR review) is the expensive one.**

### Living Documentation (Martraire)

[Cyrille Martraire's *Living Documentation*](https://www.amazon.com/Living-Documentation-Cyrille-Martraire/dp/0134689321) (480 pp, 2017–19) borrows from DDD and proposes that documentation be **generated from enriched code**. **No empirical effect-size studies appear in public databases.** The argument is craft + practitioner reports, not controlled trials.

The OpenSpec model aligns with Martraire's: specs aren't a separate artifact, they're *the* artifact — generated changes are derived from them; code references them.

### Apprenticeship / mentoring

- **["A case study of implicit mentoring in Apache" (ESEC/FSE 2022)](https://www.researchgate.net/publication/365285143_A_case_study_of_implicit_mentoring_its_prevalence_and_impact_in_Apache)** — quantifies implicit mentoring via PR comments across 37 Apache projects, builds an NLP classifier. Strongest empirical evidence that **PR-level review IS mentoring** in OSS.
- **[The Science of Mentoring Relationships (NCBI)](https://www.ncbi.nlm.nih.gov/books/NBK552775/)** — STEMM mentoring literature; useful background, not domain-specific.

Implication: **PR review on OpenSpec changes is where humans actually learn the format.** Investments in PR-review quality (review checklists, OpenSpec-specific reviewer tooling, the LLM-judge results visible inline) are higher-ROI for human training than dedicated tutorial documents.

### Carl Wieman-style apprenticeship / deliberate practice

The Wieman pedagogy (active learning, deliberate practice) doesn't have direct technical-writing analogs in the literature, but the *pattern* — small, scaffolded exercises with immediate expert feedback — maps onto: **give a new author a Forge-dispatched draft, ask them to fix it against the rubric, repeat.** This is the human side of the GEPA loop.

---

## Layer 8: AI-specific risks for spec authoring

### Spec poisoning / adversarial documents

[PoisonedRAG (Zou et al., USENIX Security 2025)](https://www.usenix.org/system/files/usenixsecurity25-zou-poisonedrag.pdf) is the seminal result:
- Just **5 carefully crafted documents** can manipulate RAG responses with **97% ASR on NQ, 99% on HotpotQA, 91% on MS-MARCO** against PaLM-2.
- Black-box, query-agnostic attack works.

The threat model translates to OpenSpec: **if Forge / Muse / Maestro retrieve from `openspec/` and a hostile (or careless) PR introduces a poisoned spec, the orchestra can be steered.** Mitigations from [OWASP LLM01:2025](https://genai.owasp.org/llmrisk/llm01-prompt-injection/) and the [ByteArmor 2025 defense guide](https://bytearmor.ai/blog/prompt-injection-data-poisoning):
- Treat `openspec/` mutations as **privileged operations** — gate behind human review (already true for `current/`, the policy that direct edits aren't allowed is exactly the right control).
- Add a poisoning detection pass: [RevPRAG](https://arxiv.org/html/2411.18948v1) reports 98%+ detection accuracy on three poisoning datasets via LLM activation analysis. Probably overkill at our scale; the **structural mitigation (PR review + LLM-judge for newly added specs)** does most of the work.

### Sycophancy in spec review

The [SycEval framework](https://arxiv.org/abs/2502.08177) is the canonical 2025 evaluation:
- 58.19% overall sycophancy rate. Gemini 62.47% (highest), GPT-4o 56.71% (lowest), Claude in between.
- **Progressive sycophancy** (agrees toward the right answer): 43.52%. **Regressive sycophancy** (agrees toward the wrong answer): 14.66%.
- [Confirmation bias in LLM code review](https://arxiv.org/html/2509.21305v1) — same pattern in code-review settings.

For OpenSpec: **a reviewer LLM that always agrees with the author LLM is worse than no reviewer.** Two structural mitigations:
1. **Cross-family judging** — author with Sonnet, judge with Gemini/GPT-5 or vice versa. Reduces preference-leakage.
2. **Adversarial rubric** — explicit "find one weakness or output a hard pass" instruction. SycEval-style prompts that ask "is this complete?" without explicit critique requirement skew agreeable.

### Drift detection

[InsightFinder 2025 drift guide](https://insightfinder.com/blog/hidden-cost-llm-drift-detection/), [arxiv 2511.07585 LLM Output Drift (2025)](https://arxiv.org/abs/2511.07585), [orq.ai drift](https://orq.ai/blog/model-vs-data-drift). Two kinds of drift to watch in spec authoring:
- **Style drift** — across weeks, generated specs gradually deviate from the OpenSpec house style. Detect via: longitudinal Vale-rule violation rate, average sentence length, vocabulary overlap with `current/` corpus.
- **Semantic drift** — embedding-space shift between current-month outputs and the canonical corpus.

[Giskard](https://github.com/Giskard-AI/giskard) is an OSS option for scheduled drift tests. Lightweight version: a weekly cron that runs the eval set and posts the delta to a dashboard.

### Reproducibility / determinism

- **Temperature = 0 ≠ deterministic.** Mixture-of-Experts routing + batch context + floating-point order make modern inference inherently non-deterministic ([Vincent Schmalbach](https://www.vincentschmalbach.com/does-temperature-0-guarantee-deterministic-llm-outputs/), [arxiv 2408.04667 "Non-Determinism of Deterministic LLM Settings"](https://arxiv.org/html/2408.04667v5)).
- **Seed parameter** only helps in self-hosted (vLLM) on **same hardware + same version** — not on hosted APIs.
- **Practical pattern**: idempotent caching by `(model, prompt_hash, params_hash)` for "same-result-on-same-input" semantics — see [Keywords AI 2025 consistency guide](https://www.keywordsai.co/blog/llm_consistency_2025).

For OpenSpec: **don't promise spec-generation reproducibility.** Promise rubric-graded quality. Quality you can measure and gate on; bit-exact reproducibility you cannot.

### Authoring vs. reading task asymmetry

I could not find a dedicated study on whether LLMs hallucinate *more* when authoring vs reading the same text. The 2025 hallucination surveys ([Frontiers 2025 survey](https://www.frontiersin.org/journals/artificial-intelligence/articles/10.3389/frai.2025.1622292/full), [arxiv 2510.06265](https://arxiv.org/html/2510.06265v2), [ACM 3703155](https://dl.acm.org/doi/10.1145/3703155)) taxonomize **factuality** vs **faithfulness** hallucinations, but don't decompose by task direction. **Open question for our domain — could be measured with the same golden set we'd build for Layer 5.**

---

## Synthesis — research roadmap for Rapoport Studio

### Three cheapest wins (do this quarter — Q3 2026)

1. **Vale on `openspec/` in CI.** Custom YAML rules for OpenSpec terminology (capability, change, archive, etc.), readability target = grade 10–12, sentence-shape rules for the prose-style proposal format. Two days of work, immediate per-PR signal. Wins because it's deterministic, fast, and per-spec-rule.
2. **30–50 graded OpenSpec examples as a golden set.** One person, two days, one capability area (start with `platform.md` or `forge.md`). Becomes the gradient signal for *everything* downstream — Layer-4 GEPA needs it, Layer-5 eval needs it, Layer-1 LLM-judge calibration needs it. Single highest-leverage artifact in this whole document.
3. **Cross-family LLM-judge with explicit rubric.** Use Sonnet to author, Gemini-2.5 or GPT-5 to judge. Rubric = the seven OpenSpec principles + IEEE-830 element score (completeness, correctness, preciseness, consistency). Run on every spec PR. Mitigates sycophancy and preference-leakage in one move.

### Three medium investments (next 6 months — Q4 2026–Q1 2027)

1. **DSPy + GEPA + BAML compile loop** for the spec-author role. Production case study shows +20 pp / ~$3 / ~1200 rollouts. Needs the golden set (#2 above) as ground truth. Output: optimized author + critic prompts that live in the L2 (skill) cache layer.
2. **Braintrust eval harness in CI.** Wire the golden set + a small RAG-style retrieval test (when the corpus crosses 200 K tokens) to PR checks. Weekly cron for drift detection (Vale-rule rate + embedding-space distance from `current/`).
3. **CoVe pass on every generated spec.** Draft → verify against `current/<capability>.md` claims → revise. Capped at 2 iterations (per the diminishing-returns literature). Modest token cost, measurable hallucination reduction (FactScore 55.9 → 71.4 in CoVe's paper).

### Three long bets (12–18 months — Q2 2027 onward)

1. **Publish the "OpenSpec quality benchmark."** 200–500 graded prose-style spec examples, public on HuggingFace, paired with a leaderboard. There is no equivalent today (RESpecBench covers *formal* specs; QuARS/Smella cover *requirements smells*). Modest research artifact, high signaling value for Rapoport as a methodology shop, and gives us our own evaluation moat.
2. **Hybrid retrieval if/when `openspec/` exceeds 200 K tokens.** BGE-M3 or Voyage-code-3 embeddings into pgvector (already in stack via Supabase) + BM25 via Postgres `ts_vector` + RRF. Skip rerankers below 1k docs. Don't build before the threshold.
3. **Spec-poisoning red-team and detector.** PoisonedRAG-style adversarial PRs against an OpenSpec checkout; measure orchestra resistance; ship a lightweight RevPRAG-style activation detector if structural controls (PR review + LLM-judge) prove insufficient. Risk-reduction work — invest when the orchestra is autonomous enough that the PR-review gate is bypassable.

---

## Citation index

Anchored URLs only. Marked `[unverified]` if no primary source could be confirmed.

- Vale: [vale.sh/docs](https://vale.sh/docs); [Datadog Vale engineering post](https://www.datadoghq.com/blog/engineering/how-we-use-vale-to-improve-our-documentation-editing-process/); [Grafana Writers' Toolkit](https://grafana.com/docs/writers-toolkit/review/lint-prose/); [Spectro 2025 post](https://www.spectrocloud.com/blog/how-we-use-vale-to-enforce-better-writing-in-docs-and-beyond)
- proselint: [github.com/amperser/proselint](https://github.com/amperser/proselint); alex: [github.com/get-alex/alex](https://github.com/get-alex/alex); write-good: [github.com/btford/write-good](https://github.com/btford/write-good); retext: [retextjs.github.io](https://retextjs.github.io); textstat / py-readability-metrics: [github.com/cdimascio/py-readability-metrics](https://github.com/cdimascio/py-readability-metrics)
- QuARS: [CEUR Vol-2376 NLP4RE19](https://ceur-ws.org/Vol-2376/NLP4RE19_paper07.pdf); Smella vs newer: [arxiv 2403.17479](https://arxiv.org/html/2403.17479); SpanBERT NER 2025: [Wiley SMR 70041](https://onlinelibrary.wiley.com/doi/10.1002/smr.70041)
- RESpecBench: [OpenReview eFwJZIN9eI](https://openreview.net/forum?id=eFwJZIN9eI)
- SycEval: [arxiv 2502.08177](https://arxiv.org/abs/2502.08177); LLM-as-Judge survey: [arxiv 2412.05579v2](https://arxiv.org/html/2412.05579v2); Wikipedia: [LLM-as-a-Judge](https://en.wikipedia.org/wiki/LLM-as-a-Judge)
- Long-context vs RAG: [Tianpan 2026-04-09](https://tianpan.co/blog/2026-04-09-long-context-vs-rag-production-decision-framework); [Open-techstack 2026](https://open-techstack.com/blog/rag-vs-long-context-2026/); [Sitepoint 1M token](https://www.sitepoint.com/long-context-vs-rag-1m-token-windows/); [Databricks long-context RAG benchmark](https://www.databricks.com/blog/long-context-rag-performance-llms)
- Lost in the Middle: [TACL 2024](https://aclanthology.org/2024.tacl-1.9/); [arxiv 2307.03172](https://arxiv.org/abs/2307.03172)
- BGE-M3: [HuggingFace BAAI/bge-m3](https://huggingface.co/BAAI/bge-m3); [BGE docs](https://bge-model.com/bge/bge_m3.html)
- Voyage code-3: [Voyage blog 2024-12-04](https://blog.voyageai.com/2024/12/04/voyage-code-3/); [Voyage-3-large 2025-01-07](https://blog.voyageai.com/2025/01/07/voyage-3-large/); [MongoDB blog](https://www.mongodb.com/company/blog/voyage-code-3-more-accurate-code-retrieval-lower-dimensional-quantized-embeddings); [Modal benchmark](https://modal.com/blog/6-best-code-embedding-models-compared)
- Hybrid retrieval: [Supermemory 2026-04](https://blog.supermemory.ai/hybrid-search-guide/); [Glaforge RRF 2026-02](https://glaforge.dev/posts/2026/02/10/advanced-rag-understanding-reciprocal-rank-fusion-in-hybrid-search/); [Ranjan Kumar full-stack hybrid](https://ranjankumar.in/building-a-full-stack-hybrid-search-system-bm25-vectors-cross-encoders-with-docker)
- Late chunking: [Jina blog](https://jina.ai/news/late-chunking-in-long-context-embedding-models/); [arxiv 2409.04701v3](https://arxiv.org/pdf/2409.04701); [github.com/jina-ai/late-chunking](https://github.com/jina-ai/late-chunking)
- pgvector / LanceDB: [CallSphere 2026 benchmark](https://callsphere.ai/blog/vector-database-benchmarks-2026-pgvector-qdrant-weaviate-milvus-lancedb); [4xxi comparison](https://4xxi.com/articles/vector-database-comparison/); [Zilliz pgvector vs LanceDB](https://zilliz.com/comparison/pgvector-vs-lancedb)
- Anthropic prompt caching: [official docs](https://platform.claude.com/docs/en/build-with-claude/prompt-caching); [Spring AI 2025-10-27](https://spring.io/blog/2025/10/27/spring-ai-anthropic-prompt-caching-blog/); [Finout 2026 pricing guide](https://www.finout.io/blog/anthropic-api-pricing)
- KV-cache-aware prompt engineering: [F5 Ankit Bko 2025-08](https://ankitbko.github.io/blog/2025/08/prompt-engineering-kv-cache/); [LMCache Claude Code 2025-12-23](https://blog.lmcache.ai/en/2025/12/23/context-engineering-reuse-pattern-under-the-hood-of-claude-code/)
- GPTCache: [github.com/zilliztech/GPTCache](https://github.com/zilliztech/GPTCache); [paper aclanthology.org/2023.nlposs-1.24](https://aclanthology.org/2023.nlposs-1.24.pdf)
- GEPA: [arxiv 2507.19457](https://arxiv.org/abs/2507.19457); [dspy.ai GEPA docs](https://dspy.ai/api/optimizers/GEPA/overview/); [Arize benchmark](https://arize.com/blog/gepa-vs-prompt-learning-benchmarking-different-prompt-optimization-approaches/)
- DSPy + BAML production: [Kevin Madura kmad.ai](https://kmad.ai/DSPy-Optimization); [Vicente Reig BAML schema](https://vicentereig.github.io/dspy.rb/blog/articles/baml-schema-format/); [Prashanth Rao micro-benchmark](https://github.com/prrao87/structured-outputs)
- Reflexion: [Shinn et al. Semantic Scholar](https://www.semanticscholar.org/paper/Reflexion:-an-autonomous-agent-with-dynamic-memory-Shinn-Labash/46299fee72ca833337b3882ae1d8316f44b32b3c); MAR: [arxiv 2512.20845](https://arxiv.org/html/2512.20845); Long-horizon: [arxiv 2509.09677v1](https://arxiv.org/html/2509.09677v1)
- CoVe: [aclanthology 2024.findings-acl.212](https://aclanthology.org/2024.findings-acl.212/); [arxiv 2309.11495](https://arxiv.org/abs/2309.11495)
- Constitutional AI: [Anthropic CAI paper](https://www-cdn.anthropic.com/7512771452629584566b6303311496c262da1006/Anthropic_ConstitutionalAI_v2.pdf); [Anthropic 2025 new constitution](https://www.anthropic.com/news/claude-new-constitution); [TIME 2025](https://time.com/7354738/claude-constitution-ai-alignment/)
- AutoResearch: [latent.space autoresearch](https://www.latent.space/p/ainews-autoresearch-sparks-of-recursive); [arxiv 2603.07300 AutoResearch-RL](https://arxiv.org/abs/2603.07300); [arxiv 2603.23420 Bilevel](https://arxiv.org/html/2603.23420); [arxiv 2604.01007 OmniMem](https://arxiv.org/html/2604.01007v1)
- Eval platforms: [Braintrust Langfuse alternatives 2026](https://www.braintrust.dev/articles/langfuse-alternatives-2026); [Laminar Braintrust alternatives 2026](https://laminar.sh/article/braintrust-alternatives-2026); [Softcery 2025 platforms](https://softcery.com/lab/top-8-observability-platforms-for-ai-agents-in-2025)
- Eval frameworks: [DeepEval](https://github.com/confident-ai/deepeval); [Ragas in DeepEval](https://deepeval.com/docs/metrics-ragas); [promptfoo vs DeepEval vs RAGAS](https://genai.qa/blog/promptfoo-vs-deepeval-vs-ragas/); [Inference.net 2026 comparison](https://inference.net/content/llm-evaluation-tools-comparison/)
- Golden dataset: [Microsoft promptflow guidance](https://github.com/microsoft/promptflow-resource-hub/blob/main/sample_gallery/golden_dataset/copilot-golden-dataset-creation-guidance.md); [Maxim](https://www.getmaxim.ai/articles/building-a-golden-dataset-for-ai-evaluation-a-step-by-step-guide/); [Statsig](https://www.statsig.com/perspectives/golden-datasets-evaluation-standards); [Musubi content-mod](https://www.musubilabs.ai/post/how-to-build-golden-datasets-for-content-moderation); [Dextralabs prod-RAG 2025](https://dextralabs.com/blog/production-rag-in-2025-evaluation-cicd-observability/)
- Cody / Cursor / Continue: [Augment Cursor vs Cody embeddings](https://www.augmentcode.com/tools/cursor-vs-sourcegraph-cody-embeddings-and-monorepo-scale); [Sourcegraph anatomy of an AI coding assistant](https://sourcegraph.com/blog/anatomy-of-a-coding-assistant); [Sourcegraph how Cody understands codebase](https://sourcegraph.com/blog/how-cody-understands-your-codebase)
- Diataxis: [diataxis.fr](https://diataxis.fr/); [BSSW review](https://bssw.io/items/diataxis-a-systematic-approach-to-technical-documentation-authoring); [Bernard 2024](https://emmanuelbernard.com/blog/2024/12/19/diataxis/); [Python.org discussion](https://discuss.python.org/t/adopting-the-diataxis-framework-for-python-documentation/15072)
- Cognitive Load: [Sweller 2011 chapter](https://www.emrahakman.com/wp-content/uploads/2024/10/Cognitive-Load-Sweller-2011.pdf); [Sweller 2024 ScienceDirect](https://www.sciencedirect.com/science/article/pii/S1041608024000165)
- ADR studies: [arxiv 2604.27333 One Size Fits All](https://arxiv.org/abs/2604.27333); [Architecture Decision Records in Practice ECSA 2024](https://rebekkaa.github.io/files/2024_ECSA.pdf); [arxiv 2403.01709 Can LLMs Generate ADRs](https://arxiv.org/html/2403.01709); [adr.github.io](https://adr.github.io/)
- Apprenticeship: [Apache implicit mentoring ESEC/FSE 2022](https://www.researchgate.net/publication/365285143_A_case_study_of_implicit_mentoring_its_prevalence_and_impact_in_Apache); [NCBI STEMM mentoring](https://www.ncbi.nlm.nih.gov/books/NBK552775/)
- Living Documentation: [Martraire book](https://www.amazon.com/Living-Documentation-Cyrille-Martraire/dp/0134689321); [InfoQ Q&A](https://www.infoq.com/articles/book-review-living-documentation/) — *empirical effect-size studies not located.*
- PoisonedRAG: [USENIX Security 2025](https://www.usenix.org/system/files/usenixsecurity25-zou-poisonedrag.pdf); [Poison-RAG recommender ECIR](https://link.springer.com/content/pdf/10.1007/978-3-031-88717-8_18.pdf); [Promptfoo RAG poisoning](https://www.promptfoo.dev/docs/red-team/plugins/rag-poisoning/)
- OWASP LLM01 prompt injection: [genai.owasp.org/llmrisk/llm01-prompt-injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/); [OWASP cheatsheet](https://cheatsheetseries.owasp.org/cheatsheets/LLM_Prompt_Injection_Prevention_Cheat_Sheet.html); [ByteArmor 2025 defense](https://bytearmor.ai/blog/prompt-injection-data-poisoning)
- RevPRAG detector: [arxiv 2411.18948v1](https://arxiv.org/html/2411.18948v1)
- Sycophancy: [SycEval arxiv 2502.08177](https://arxiv.org/abs/2502.08177); [Sycophancy Is Not One Thing arxiv 2509.21305](https://arxiv.org/html/2509.21305v1); [Sycophancy causes & mitigations arxiv 2411.15287](https://arxiv.org/abs/2411.15287)
- Drift detection: [InsightFinder drift](https://insightfinder.com/blog/hidden-cost-llm-drift-detection/); [arxiv 2511.07585 LLM Output Drift](https://arxiv.org/abs/2511.07585); [Stack Pulsar 2026](https://stackpulsar.com/blog/llm-model-drift-detection/); [Giskard OSS](https://github.com/Giskard-AI/giskard) `[unverified link — Giskard repo URL not opened in this pass]`
- Determinism: [Vincent Schmalbach](https://www.vincentschmalbach.com/does-temperature-0-guarantee-deterministic-llm-outputs/); [arxiv 2408.04667v5 Non-Determinism of Deterministic LLM Settings](https://arxiv.org/html/2408.04667v5); [Keywords AI 2025 consistency](https://www.keywordsai.co/blog/llm_consistency_2025); [vLLM reproducibility](https://docs.vllm.ai/en/latest/usage/reproducibility/)
- Hallucination surveys: [Frontiers 2025](https://www.frontiersin.org/journals/artificial-intelligence/articles/10.3389/frai.2025.1622292/full); [arxiv 2510.06265 comprehensive survey](https://arxiv.org/html/2510.06265v2); [ACM TOIS 3703155](https://dl.acm.org/doi/10.1145/3703155); [arxiv 2401.11817 Hallucination is Inevitable](https://arxiv.org/abs/2401.11817)
