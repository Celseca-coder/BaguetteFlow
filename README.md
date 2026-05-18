<h1 align="center">BaguetteFlow</h1>

<div align="center">

📄 **[Tech Report (Coming Soon)]()** |
🏆 **[#1 on OpenAI MLE-Bench Leaderboard](https://github.com/openai/mle-bench/blob/main/README.md)**

</div>

![BaguetteFlow](architecture.png)

## Timeline

- **[18 May 2026]** BaguetteFlow achieves **Rank-1** on OpenAI's MLE-Bench leaderboard with **65.33% All Medal Rate** using Claude Sonnet 4.6 — the highest any single-model agent has ever scored on this benchmark.
- **[Mar 2026]** Developed **BaguetteFlow**: a multi-paradigm evolutionary agentic framework for autonomous ML engineering, extending and radically re-architecting the AIDE codebase with FWA, UCT-based search, two-tier memory, and RAG-augmented debugging.

---

## Technical Details

### Introduction

BaguetteFlow is an evolutionary agentic framework for autonomous machine learning engineering. It recasts ML solution search as a **directed acyclic graph (DAG) traversal problem**, where each node encodes a natural-language reasoning plan, an executable Python solution, and an empirical validation metric. Edges represent parent–child derivation relationships: draft, improve, debug, explode, or merge operations each produce child nodes that inherit and extend their parent's lineage.

At its core, BaguetteFlow augments the classic AIDE tree-search paradigm with a **Fireworks Algorithm (FWA)** explosion mechanism, UCT-guided probabilistic action selection, a two-tier persistent memory system, and a BM25/FAISS-powered Retrieval-Augmented Debugging (RAD) engine. This combination enables BaguetteFlow to simultaneously explore diverse solution trajectories, exploit the best-performing branches through fine-grained refinement, and recover from failure modes through memory-guided debugging — all within a single 24-hour compute budget.

---

### 🌐 DAG-Based Solution Search

BaguetteFlow maintains a growing directed acyclic graph of solution nodes. Every node stores a full reasoning trace (plan), executable ML code, and an evaluated metric. Multiple solution chains coexist in the graph, allowing BaguetteFlow to preserve failed branches as historical knowledge, resume dormant lines of exploration, and fuse insights across divergent trajectories. This graph structure enables fine-grained credit assignment across multi-generation improvement chains — something flat population-based methods fundamentally cannot do.

---

### 🎆 Fireworks Algorithm (FWA) Explosion

The heart of BaguetteFlow's exploration mechanism is FWA-style explosion. When a solution node is selected for exploration, the system **fires multiple sparks in parallel** (via `ThreadPoolExecutor`), each biased toward a distinct *diversity angle* — data augmentation, architecture modification, training strategy, feature engineering, ensemble methods, loss function design, hyperparameter search, or transfer learning. The number of sparks and the explosion amplitude (mutation intensity) are both dynamically computed from the node's relative quality score in the current solution population. Elite nodes receive precision-tuning sparks; weaker nodes receive radical reconstructions.

---

### 🎯 UCT-Based Probabilistic Action Selection

Action selection is governed by a probabilistic policy conditioned on the current graph state. Each non-buggy node maintains a **UCT (Upper Confidence Bound for Trees) score** combining exploitation (validated metric value) and exploration (visit count-based confidence bonus). At every step, the policy samples one of five actions:

- **Draft** — generate a new solution from scratch using the task blueprint
- **Improve** — refine a top-k node by proposing an atomic, memory-guided change
- **Debug** — diagnose and surgically fix a bug node, guided by RAG references and debug memory
- **Explode** — fan out multiple diverse mutation sparks from a high-UCT node
- **Merge** — cross-pollinate two semantically distinct high-quality solutions via evolutionary fusion

The sampling probabilities adapt dynamically: early in the run, exploration (explode) is favored; late in the run, exploitation (merge, improve) takes precedence. The debug probability also scales with the rolling bug rate of recent nodes.

---

### 🔥 Forced Explode: Anti-Stagnation Mechanism

BaguetteFlow monitors for **metric stagnation** — runs of non-buggy improvement attempts that fail to meaningfully advance the best score. When the stagnation counter exceeds a configurable patience threshold, a *Forced Explode* is triggered at elevated temperature with `use_diff=False`, compelling the model to perform radical architectural redesign. The cooldown mechanism ensures explosions don't fire too frequently, and the stagnation counter resets upon any successful improvement.

---

### 🧠 Two-Tier Persistent Memory System

BaguetteFlow maintains two complementary memory structures:

1. **Global Deduplication Memory** — a hash-based store that fingerprints every generated node and rejects duplicate code generations, ensuring the search space is never wastefully revisited.
2. **Dual-Channel Agent Memory** — a `AgentMemoryManager` combining:
   - *Persistent Memory (MD)*: A high-level, LLM-synthesized summary of proven synergies (positive guidance) and fatal pitfalls (negative constraints) distilled from the entire run history.
   - *Recent Buffer (JSON)*: A rolling log of the most recent micro-experiments, their metrics, and outcomes — providing fine-grained local context for the next generation.

During `improve` and `explode` steps, this fused memory context is injected into every prompt, forcing the model to avoid blind spots, leverage discovered synergies, and propose genuinely novel strategies.

---

### 🔍 RAG-Augmented Debugging

The `_debug` agent employs a **BM25/FAISS-based NodeRetriever** to fetch the top-k most similar historical nodes by code/plan embedding similarity. Each retrieved reference provides the past bug type, fix plan, and key code snippet — acting as a grounded template for the current repair. Alongside the RAG context, a *debug memory* channel surfaces accumulated failure modes and invalid hypotheses, forming a two-pronged "do not repeat" guard. For shallow bugs (depth ≤ 3), surgical SEARCH/REPLACE diffs are preferred; for deeply stuck nodes, full code rewriting is used.

---

### 🔀 Evolutionary Merge (Crossover)

The `_merge` operation implements an **evolutionary crossover** strategy: given the global best node and a semantically diverse partner selected via softmax-weighted UCT sampling, BaguetteFlow prompts the LLM to (1) identify the stronger base solution, (2) extract high-value "donor genes" from the partner, and (3) graft them into the base while resolving conflicts. This late-run strategy combines the exploitation depth of focused improvement with the diversity benefit of cross-trajectory knowledge transfer.

---

### ✂️ Diff-Patch Code Modification

Rather than regenerating full scripts on every step, BaguetteFlow supports **SEARCH/REPLACE diff patching** — a token-efficient mechanism where the LLM outputs only the exact code blocks that need to change. This reduces generation cost, preserves well-performing code regions, and makes changes more interpretable. Diff mode is used by default for improve and shallow debug; full-code mode is reserved for radical mutations and deep debugging.

---

## Performance Metrics

BaguetteFlow achieves state-of-the-art performance on the OpenAI MLE-Bench benchmark, ranking **#1 on the overall leaderboard** as of May 2026. Running on a 24-hour compute budget with Claude Sonnet 4.6, BaguetteFlow attains a **65.33% All Medal Rate**, outperforming all prior single-model agents including Famou-Agent 2.0, AIBuildAI, and MARS+ across Low/Lite and Medium splits.The 19 gold medals span domains including medical imaging, audio classification, NLP, tabular regression, and scientific discovery — evidence of the framework's generalization across heterogeneous ML task types.

### MLE-Bench Leaderboard (as of May 2026)

| Agent | LLM | Low/Lite (%) | Medium (%) | High (%) | All (%) | Time |
|-------|-----|-------------|------------|----------|---------|------|
| **BaguetteFlow** 🥇 | Claude-Sonnet-4.6 | **81.82** ± 0.00 | **65.79** ± 1.52 | 40.00 ± 0.00 | **65.33** ± 0.77 | 24h |
| Famou-Agent 2.0 | Gemini-3-Pro-Preview | 80.30 ± 1.52 | 64.04 ± 2.32 | 42.22 ± 2.22 | 64.44 ± 1.18 | 24h |
| AIBuildAI | Claude-Opus-4.6 | 77.27 ± 0.00 | 61.40 ± 0.88 | 46.67 ± 0.00 | 63.11 ± 0.44 | 24h |
| CAIR MARS+ | Gemini-3-Pro-Preview | 78.79 ± 1.52 | 60.53 ± 1.52 | 44.44 ± 2.22 | 62.67 ± 0.77 | 24h |
| MLEvolve | Gemini-3-Pro-Preview | 80.30 ± 1.52 | 57.89 ± 1.52 | 42.22 ± 2.22 | 61.33 ± 1.33 | 12h |
| PiEvolve | Gemini-3-Pro-Preview | 80.30 ± 1.52 | 58.77 ± 0.88 | 40.00 ± 0.00 | 61.33 ± 0.77 | 24h |

### Medal Breakdown 

| Metric | Count |
|--------|-------|
| Total runs | 50 |
| Valid submissions | 50 |
| Any medal | 50 |
| 🥇 Gold medals | 19 |
| 🥈 Silver medals | 16 |
| 🥉 Bronze medals | 15 |
| Above median | 50|

**Selected Gold Medal Competitions:** `histopathologic-cancer-detection` · `mlsp-2013-birds` · `detecting-insults-in-social-commentary` · `predict-volcanic-eruptions-ingv-oe` · `tensorflow-speech-recognition-challenge` · `seti-breakthrough-listen` · `iwildcam-2019-fgvc6` · `dogs-vs-cats-redux-kernels-edition` · `denoising-dirty-documents` · `lmsys-chatbot-arena` · `plant-pathology-2021-fgvc8` · `learning-agency-lab-automated-essay-scoring-2` · `plant-pathology-2020-fgvc7` · `tabular-playground-series-dec-2021` · `tabular-playground-series-may-2022` · `stanford-covid-vaccine` · `aerial-cactus-identification` · `vinbigdata-chest-xray-abnormalities-detection` · `the-icml-2013-whale-challenge-right-whale-redux`

---

## Citation

If you use BaguetteFlow in your research, please cite:

```bibtex
@misc{BaguetteFlow2026,
  title   = {BaguetteFlow: Evolutionary Agentic Framework for Autonomous ML Engineering},
  author  = {AgentSAIS Team},
  year    = {2026},
  url     = {https://github.com/Celseca-coder/BaguetteFlow}
}
```

---

## About AgentSAIS

The AgentSAIS team develops next-generation autonomous ML engineering agents, focusing on evolutionary search, persistent memory, and multi-agent collaboration for long-horizon scientific discovery tasks.

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

## Contact

- GitHub: [Celseca-coder/BaguetteFlow](https://github.com/Celseca-coder/BaguetteFlow)
- Issues: [Submit an Issue](https://github.com/Celseca-coder/BaguetteFlow/issues)

---
