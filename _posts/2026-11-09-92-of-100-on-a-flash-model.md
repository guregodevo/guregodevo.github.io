---
layout: post
title: "92 of the 100 hardest LiveCodeBench problems on a Flash model"
date: 2026-11-09
---

> A cheap Flash model,<br>
> ninety-two of a hundred,<br>
> a tenth of the bill.

On October 31 I posted [five Qwen3.8-27B agents against Fable 5](https://news.ycombinator.com/item?id=50320995) on ten LiveCodeBench problems. The main feedback: ten is not enough. So this time it is all 100 of [GVS5H](https://github.com/slee-persis/GVS5H)'s hardest LiveCodeBench problems. I ran [team-solve-fanout](https://github.com/guregodevo/memdoor/tree/main/workflows/team-solve-fanout), Memdoor's team mode with three attempts in parallel, once on DeepSeek V4.1 Flash. Every test was graded, public and hidden.

| | solved | $ for the 100 | $ per solved |
|---|---|---|---|
| GVS5H manager, Qwen3.8-Flash-Next | 93.0% | 5.76 | 0.062 |
| GVS5H manager, Qwen3.8-27B | 92.4% | 47.82 | 0.52 |
| **Memdoor team-solve-fanout, DeepSeek V4.1 Flash** | **92%** | **5.49** | **0.060** |
| GVS5H manager, DeepSeek V4.1 Flash | 92.0% | 15.43 | 0.17 |
| Fable 5, single call | 90.4% | 61.11 | 0.68 |

**Fable 5's score on a Flash model, for 1/11 of its bill.**

- The GVS5H rows are the paper's: means of five passes, priced at list rates. Mine is one pass, priced at what OpenRouter billed, cache included.
- On the same model and the same prices, the paper's harness and mine cost about the same. The gap with Fable 5 is the model's price; the harness matches the paper's.
- Grading follows the paper's rule for the problems that accept more than one answer. The stock grader gives 91.

[The ranking with every row](https://github.com/guregodevo/memdoor/tree/main/examples/workflows/team-solve/bench#ranking-on-the-100) · [Per problem](https://github.com/guregodevo/memdoor/blob/main/examples/workflows/team-solve/bench/results/team-solve-fanout-lcb.tsv) · [The 8 misses, and what is next](https://github.com/guregodevo/memdoor/issues/234) · [Memdoor](https://memdoor.ai)
