---
layout: post
title: "What I learned running SWE-bench with a team of DeepSeek V4.1 Flash agents"
date: 2026-11-07
---

> It's the journey.<br>
> The start and end are spirit creations.

After I posted [a team of five Qwen3.8-27B models against Fable 5 on LiveCodeBench Hard](https://news.ycombinator.com/item?id=50320995) on Hacker News, I got a few pieces of feedback. The main one was that I should show the results on real tasks. The second was that ten problems are not enough, and I should run around a thousand so the results are significant.

So this time it is real GitHub issues: all 500 of [SWE-bench Verified](https://www.swebench.com), each one both ways, 1,000 runs. DeepSeek is one of the most popular models on OpenRouter, so I tried DeepSeek V4.1 Flash with [team-solve](https://github.com/guregodevo/memdoor/tree/main/workflows/team-solve), now built into [Memdoor](https://github.com/guregodevo/memdoor)'s coding agent as team mode. Same coding agent and the same team-solve workflow; a different benchmark and a different model. The baseline is a single call: one agent turn on the same model.

I did not expect a model like DeepSeek to gain much over a single call. The point was to measure what the harness adds and what it costs. Every prediction, test log and trace is in [guregodevo/swe-bench](https://github.com/guregodevo/swe-bench).

| | resolved | $ per issue | median time |
|---|---|---|---|
| single call (the baseline, a separate run) | 388 / 500 (77.6%) | 0.012 | 91 s |
| team-solve | **400** / 500 (80.0%) | 0.022 | 258 s |
| team-solve's own single call (its first step) | 393 / 500 (78.6%) | | |

**The takeaway: 3.1% more issues resolved for one cent more per task ($0.022 instead of $0.012) on a Flash model.** That is 400 issues instead of 388, from 77.6% to 80.0%.

**team-solve starts with its own single call and keeps it unless the team does better. It never lost an issue that single call had solved: 7 gained, 0 lost (p = 0.016). On the 324 issues I never looked at while building team-solve, 5 gained and 0 lost.**

The baseline is a separate single call, and the same model does not solve the same issue the same way twice: the baseline and team-solve's own single call disagree on 47 issues. That noise makes the comparison with the baseline weaker:

| | issues | team-solve against its own single call | p | team-solve against the baseline | p |
|---|---|---|---|---|---|
| all | 500 | 7 gained, 0 lost | 0.016 | 27 gained, 15 lost | 0.09 |
| seen while building team-solve | 176 | 2 gained, 0 lost | 0.5 | 14 gained, 4 lost | 0.03 |
| never seen | 324 | 5 gained, 0 lost | 0.06 | 13 gained, 11 lost | 0.84 |

Before this run I ran pilots on 172 Verified issues and changed team-solve after watching them: the step that keeps a single call the team cannot beat, how a check that already fails is counted, a crash on non-ASCII prompts. With the ten its README names, that is 176 issues, mostly the first in the dataset's order ([the list](https://github.com/guregodevo/swe-bench/blob/main/kit/seen-ids.txt)).

## When the single call fails

team-solve's own single call failed on 107 issues. What happened next:

| the single call's patch | issues | team-solve fixed |
|---|---|---|
| failed its check, so the team's agents took over | 29 | 7 |
| wrong, but its check passed, so the team never ran | 69 | 0 |
| unfinished: the single call ran past its 30-minute limit and the run stopped there | 9 | 0 |

The team fixes 7 of the 29 it works on, 5 of 21 on the issues I never looked at. The 69 are the gap: the project's own tests passed a wrong fix, because SWE-bench grades with tests written for the fix, and the team never saw them. The 9 are a bug: a single call that runs out of time should hand over to the team, not end the run. Both are filed, with every issue id: [#150](https://github.com/guregodevo/memdoor/issues/150), [#151](https://github.com/guregodevo/memdoor/issues/151), [#152](https://github.com/guregodevo/memdoor/issues/152).

## The data

Issue by issue:

| | issues |
|---|---|
| both resolved | 373 |
| only team-solve | 27 |
| only the single call | 15 |
| neither | 85 |

27 against 15 is p = 0.09. On every one of the 15, team-solve's own single call failed too: the baseline's separate run was right where team-solve's was not.

By repository:

| repository | issues | single call | team-solve |
|---|---|---|---|
| django | 231 | 183 | 195 |
| sympy | 75 | 58 | 60 |
| sphinx | 44 | 33 | 30 |
| matplotlib | 34 | 26 | 27 |
| scikit-learn | 32 | 31 | 31 |
| astropy | 22 | 15 | 16 |
| xarray | 22 | 17 | 16 |
| pytest | 19 | 18 | 16 |
| pylint | 10 | 3 | 4 |
| requests | 8 | 3 | 4 |
| seaborn | 2 | 0 | 0 |
| flask | 1 | 1 | 1 |

By SWE-bench's own difficulty label:

| difficulty | issues | single call | team-solve |
|---|---|---|---|
| under 15 min | 194 | 166 | 173 |
| 15 min to 1 hour | 261 | 199 | 204 |
| 1 to 4 hours | 42 | 22 | 23 |
| over 4 hours | 3 | 1 | 0 |

By the year the issue was filed:

| year | issues | single call | team-solve |
|---|---|---|---|
| 2013–2016 | 8 | 3 | 3 |
| 2017 | 16 | 15 | 15 |
| 2018 | 24 | 18 | 20 |
| 2019 | 98 | 81 | 86 |
| 2020 | 108 | 84 | 88 |
| 2021 | 86 | 63 | 62 |
| 2022 | 102 | 81 | 83 |
| 2023 | 58 | 43 | 43 |

## The cost

From OpenRouter's own charge on each call, over the 1,000 runs that count:

| | single call | team-solve |
|---|---|---|
| total for the 500 issues | $6.01 | $10.84 |
| per issue, mean | $0.012 | $0.022 |
| per issue, median | $0.004 | $0.012 |
| per issue, 90th percentile | $0.038 | $0.053 |
| most expensive issue | $0.12 | $0.27 |
| per resolved issue | $0.016 | $0.027 |
| model calls per issue | 28 | 36 |
| input tokens | 296 M, 88% cached | 405 M, 86% cached |
| output tokens | 10.4 M | 20.3 M |
| time per issue, median | 1.5 min | 4.3 min |
| time per issue, 90th percentile | 15 min | 27 min |
| longest issue | 32 min | 2 h |

The 12 extra issues team-solve resolved cost $4.83 more, about $0.40 each. Most of the input is the same context sent again, and OpenRouter bills cached tokens at a fraction of the price.

## Check it

```
$ git clone https://github.com/guregodevo/swe-bench && cd swe-bench
$ pip install "swebench @ git+https://github.com/SWE-bench/SWE-bench@02e7a74"
$ d=20261106_memdoor-team-solve_deepseek-v4.1-flash
$ jq '.resolved | length' $d/results/results.json
400
$ swebench submit verify $d -s verified --logs $d/logs
20261106_memdoor-team-solve_deepseek-v4.1-flash: re-graded 493 instance(s)
PASS -- the entry matches its artifacts.
```

`verify` grades every issue again from its recorded test output. The 7 issues it skips have an empty patch.

One issue the single call failed and team-solve fixed:

```
$ jq '."django__django-12406".resolved' 20261106_memdoor_deepseek-v4.1-flash/logs/django__django-12406/report.json
false
$ jq '."django__django-12406".resolved' $d/logs/django__django-12406/report.json
true
```

The trace has every agent session and every file the run wrote.

## Run it

In the Memdoor window, Shift+Tab to team mode and type the task, or:

```
/w team-solve Fix the failing date parser test; go test ./... must pass
```

From a shell:

```
curl -fsSL https://memdoor.ai/install.sh | bash
memdoor workflow run team-solve "Fix the failing date parser test; go test ./... must pass"
```

## What it is not

It is not on the leaderboard. SWE-bench Verified takes submissions only from academic teams and research labs since November 2025. team-solve was built while watching 176 of these issues, split out above. Each issue ran once. Fable 5 is about 95%.

In the first run, OpenRouter's web search reached upstream fixes on 126 issues. Those 126 ran again with the web tools removed. The [run's README](https://github.com/guregodevo/swe-bench#what-went-wrong-and-what-was-run-again) has the details.

## What is next

What motivated me to build a coding agent like Memdoor is the harness. I have always believed that context engineering and a bit of DAG orchestration would make a better harness. [OpenClaw](https://github.com/openclaw/openclaw) paved the way with its multi-agent architecture. [Pi](https://github.com/earendil-works/pi) and other coding agents add layers of their own, but there is still room for improvement when it comes to complex DAGs of tasks. That is why I started to build an orchestrator, [mario](https://github.com/guregodevo/mario), and then decided to build a coding agent with it.

team-solve still needs to be ironed out, but it already works with small models. This benchmark on a Flash model is more evidence that a better harness can take a cheaper model closer to the frontier ones. That is the journey I want to share with the community.

[guregodevo/swe-bench](https://github.com/guregodevo/swe-bench) · [Results in Memdoor](https://github.com/guregodevo/memdoor/tree/main/examples/workflows/team-solve/bench/memdoor-swe-bench) · [team-solve](https://github.com/guregodevo/memdoor/tree/main/workflows/team-solve) · [Team mode](https://memdoor.ai/docs/team-mode) · [Memdoor](https://memdoor.ai)
