---
layout: post
title: "92 of the 100 hardest LiveCodeBench problems with DeepSeek V4.1 Flash: the GVS5H score, 91% cheaper than Fable 5"
date: 2026-11-09
---

> A cheap Flash model,<br>
> ninety-two of a hundred,<br>
> a tenth of the bill.

On October 31 I posted [five Qwen3.8-27B agents against Fable 5](https://news.ycombinator.com/item?id=50320995) on ten LiveCodeBench problems. The main feedback: ten is not enough. So this time it is all 100 of [GVS5H](https://github.com/slee-persis/GVS5H)'s hardest LiveCodeBench problems. I ran [team-solve-fanout](https://github.com/guregodevo/memdoor/tree/main/workflows/team-solve-fanout), Memdoor's team mode with three attempts in parallel, once on DeepSeek V4.1 Flash. Every test was graded, public and hidden.

| | solved | $ per solved | Memdoor team-solve-fanout against it |
|---|---|---|---|
| Fable 5, single call | 90.4% | 0.68 | about 1/11 the price, +1.6 points |
| GVS5H manager, Qwen3.8-27B | 92.4% | 0.52 | about 1/9 the price, level within noise |
| GVS5H manager, DeepSeek V4.1 Flash | 92.0% | 0.054 | same price, same score |
| GVS5H manager, Qwen3.8-Flash-Next | 93.0% | 0.062 | same price, 1 point lower |
| **Memdoor team-solve-fanout, DeepSeek V4.1 Flash** | **92%** | **0.060** | |

**Fable 5's score on a Flash model, for 1/11 of its bill.**

- Mine is one pass, priced at what OpenRouter billed ($5.49 for the 100). The other rows are the paper's means of five passes at list rates, except the DeepSeek row: GVS5H's own token counts (`runs/per_problem_tokens.json`) at the rates OpenRouter billed my run. At DeepSeek's list rate the paper prices that row at $0.17.
- On DeepSeek the two harnesses write the same output (12.4M and 12.3M tokens a pass). Mine reads more input (88M against 1.5M), most of it from cache. The gap with Fable 5 is the model's price.
- Grading follows the paper's rule for the problems that accept more than one answer. The stock grader gives 91.

[The ranking with every row](https://github.com/guregodevo/memdoor/tree/main/examples/workflows/team-solve/bench#ranking-on-the-100) · [Per problem](https://github.com/guregodevo/memdoor/blob/main/examples/workflows/team-solve/bench/results/team-solve-fanout-lcb.tsv) · [The 8 misses, and what is next](https://github.com/guregodevo/memdoor/issues/234) · [Memdoor](https://memdoor.ai)
