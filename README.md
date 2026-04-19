# Qwen3.x Series Benchmark Comparison

Comparative benchmark results across the Qwen3.x model family, covering Coding Agent, General Agent, Search Agent, Knowledge, Instruction Following, Long Context, STEM & Reasoning, and Multilingualism categories.

| Category | Benchmark | Qwen3.5-27B | Qwen3.6-35B-A3B | Qwen3.5-35B-A3B | Qwen3.5-122B-A10B |
|---|---|---|---|---|---|
| **Coding Agent** | SWE-bench Verified | **75.0** | 73.4 | 69.2 | 72.0 |
| | SWE-bench Multilingual | **69.3** | 67.2 | — | — |
| | SWE-bench Pro | **51.2** | 49.5 | — | — |
| | Terminal-Bench 2.0 | 41.6 | **51.5** | 40.5 | 49.4 |
| | Claw-Eval Avg | 64.3 | **68.7** | — | — |
| | Claw-Eval Pass^3 | 46.2 | **50.0** | — | — |
| | SkillsBench Avg | **527.2*** | 28.7 | — | — |
| | QwenClawBench | 52.2 | **52.6** | — | — |
| | NL2Repo | 27.3 | **29.4** | — | — |
| | QwenWebBench | 1068 | **1397** | — | — |
| | CodeForces | — | — | 2028 | **2100** |
| | OJBench | — | — | 36.0 | **39.5** |
| | FullStackBench en | — | — | 58.1 | **62.6** |
| | FullStackBench zh | — | — | 55.0 | **58.7** |
| **General Agent** | TAU3-Bench | **68.4** | 67.2 | — | — |
| | TAU2-Bench | — | — | **81.2** | 79.5 |
| | BFCL-V4 | — | — | 67.3 | **72.2** |
| | VITA-Bench | **41.8** | 35.6 | 31.9 | 33.6 |
| | DeepPlanning | 22.6 | **25.9** | 22.8 | 24.1 |
| | Tool Decathlon | **31.5** | 26.9 | — | — |
| | MCPMark | 36.3 | **37.0** | — | — |
| | MCP-Atlas | **68.4** | 62.8 | — | — |
| | WideSearch | **66.4** | 60.1 | 57.1 | 60.5 |
| **Search Agent** | HLE w/ tool | — | — | 47.4 | **47.5** |
| | Browsecomp | — | — | 61.0 | **63.8** |
| | Browsecomp-zh | — | — | 69.5 | **69.9** |
| | Seal-0 | — | — | 41.4 | **44.1** |
| **Knowledge** | MMLU-Pro | 86.1 | 85.2 | 85.3 | **86.7** |
| | MMLU-Redux | 93.2 | 93.3 | 93.3 | **94.0** |
| | SuperGPQA | 65.6 | 64.7 | 63.4 | **67.1** |
| | C-Eval | 90.5 | 90.0 | 90.2 | **91.9** |
| **Instruction Following** | IFEval | — | — | 91.9 | **93.4** |
| | IFBench | — | — | 70.2 | **76.1** |
| | MultiChallenge | — | — | 60.0 | **61.5** |
| **Long Context** | AA-LCR | — | — | 58.5 | **66.9** |
| | LongBench v2 | — | — | 59.0 | **60.2** |
| **STEM & Reasoning** | GPQA | 85.5 | 86.0 | 84.2 | **86.6** |
| | HLE | 24.3 | 21.4 | 22.4 | **25.3** |
| | LiveCodeBench v6 | **80.7** | 80.4 | 74.6 | 78.9 |
| | HMMT Feb 25 | **92.0** | 90.7 | 89.0 | 91.4 |
| | HMMT Nov 25 | 89.8 | 89.1 | 89.2 | **90.3** |
| | HMMT Feb 26 | **84.3** | 83.6 | — | — |
| | IMOAnswerBench | **79.9** | 78.9 | — | — |
| | AIME26 | 92.6 | **92.7** | — | — |
| **Multilingualism** | MMMLU | — | — | 85.2 | **86.7** |
| | MMLU-ProX | — | — | 81.0 | **82.2** |
| | NOVA-63 | — | — | 57.1 | **58.6** |
| | INCLUDE | — | — | 79.7 | **82.8** |
| | Global PIQA | — | — | 86.6 | **88.4** |
| | PolyMATH | — | — | 64.4 | **68.9** |
| | WMT24++ | — | — | 76.3 | **78.3** |
| | MAXIFE | — | — | 86.6 | **87.9** |
| **Total Wins** | | **12\*\*** | **9** | **1** | **7/29\*\*\*** |

---

\* The Qwen3.5-27B SkillsBench Avg score of **527.2** is likely not a percentage and appears to be on a fundamentally different scale than the Qwen3.6-35B-A3B score of 28.7 for the same benchmark. This could indicate a raw cumulative score, a scoring methodology change between model releases, or a data entry error in the source table. Treat this figure with caution and verify against the original SkillsBench documentation before using it in any comparative analysis.

\*\* Qwen3.5-27B's win count of 12 excludes the flagged SkillsBench Avg score. Including it brings the total to 13.

\*\*\* Qwen3.5-122B-A10B's total win count is 29, but only **7/29** of those wins occur on benchmarks also run by Qwen3.5-27B or Qwen3.6-35B-A3B (MMLU-Pro, MMLU-Redux, SuperGPQA, C-Eval, GPQA, HLE, HMMT Nov 25). The remaining 22 wins come from benchmarks run exclusively against Qwen3.5-35B-A3B — Multilingualism, Search Agent, Instruction Following, and Long Context categories — where there is no cross-model competition. Qwen3.5-35B-A3B's single win (TAU2-Bench) similarly reflects limited benchmark overlap rather than overall underperformance.

> **Note:** — indicates the benchmark was not reported for that model. Wins are only bolded where two or more models share a benchmark.

---

## Sources

### Qwen3.5-122B-A10B
- **Hugging Face:** https://huggingface.co/Qwen/Qwen3.5-122B-A10B
- **Blog:** https://qwen.ai/blog?id=qwen3.5
- **License:** Apache 2.0
- **Released:** February 2026

```bibtex
@misc{qwen3.5,
    title  = {{Qwen3.5}: Towards Native Multimodal Agents},
    author = {{Qwen Team}},
    month  = {February},
    year   = {2026},
    url    = {https://qwen.ai/blog?id=qwen3.5}
}
```

### Qwen3.6-35B-A3B
- **Hugging Face:** https://huggingface.co/Qwen/Qwen3.6-35B-A3B
- **Blog:** https://qwen.ai/blog?id=qwen3.6-35b-a3b
- **License:** Apache 2.0
- **Released:** April 2026

```bibtex
@misc{qwen36_35b_a3b,
    title  = {{Qwen3.6-35B-A3B}: Agentic Coding Power, Now Open to All},
    author = {{Qwen Team}},
    month  = {April},
    year   = {2026},
    url    = {https://qwen.ai/blog?id=qwen3.6-35b-a3b}
}
```
