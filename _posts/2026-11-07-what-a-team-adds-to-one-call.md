---
layout: post
title: "What a team adds to one call: 400 of 500 SWE-bench Verified issues at two cents each"
date: 2026-11-07
---

> One try, then a check.<br>
> When it fails, the team steps in.<br>
> Nothing it touched broke.

I ran [team-solve](https://github.com/guregodevo/memdoor/tree/main/workflows/team-solve), the workflow behind [Memdoor](https://github.com/guregodevo/memdoor)'s team mode, on all 500 issues of [SWE-bench Verified](https://www.swebench.com). The baseline is one agent turn on the same model, DeepSeek V4.1 Flash. Every prediction, test log and trace is in [guregodevo/swe-bench](https://github.com/guregodevo/swe-bench).

| | resolved | $ per issue | median time |
|---|---|---|---|
| one agent turn | 388 / 500 (77.6%) | 0.012 | 91 s |
| team-solve | **400** / 500 (80.0%) | 0.022 | 258 s |
| team-solve's first try | 385 / 500 (77.0%) | | |

The team never made its first try worse. It changed that patch on 63 issues. 15 went from failing to passing and none went the other way. Team mode cost twice as much and never lost a fix.

## The data

Against its own first try and against a separate single turn:

| team-solve against | gained | lost | p |
|---|---|---|---|
| its first try | 15 | 0 | 0.00006 |
| one agent turn | 27 | 15 | 0.09 |

Both arms resolved 373, neither resolved 85.

By repository:

| repository | issues | one turn | team-solve |
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

| difficulty | issues | one turn | team-solve |
|---|---|---|---|
| under 15 min | 194 | 166 | 173 |
| 15 min to 1 hour | 261 | 199 | 204 |
| 1 to 4 hours | 42 | 22 | 23 |
| over 4 hours | 3 | 1 | 0 |

By the year the issue was filed:

| year | issues | one turn | team-solve |
|---|---|---|---|
| 2013–2016 | 8 | 3 | 3 |
| 2017 | 16 | 15 | 15 |
| 2018 | 24 | 18 | 20 |
| 2019 | 98 | 81 | 86 |
| 2020 | 108 | 84 | 88 |
| 2021 | 86 | 63 | 62 |
| 2022 | 102 | 81 | 83 |
| 2023 | 58 | 43 | 43 |

Cost, from OpenRouter's own figure on each call: $6.01 for the 500 single turns, $10.84 for the 500 team runs.

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

One issue both the single turn and the first try failed, and the team fixed:

```
$ grep -m1 '^kept:' $d/trajs/django__django-12406.md
kept: the team's work (check 0, judge PASS; the direct attempt's check was 1)
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

It is not on the leaderboard. SWE-bench Verified takes submissions only from academic teams and research labs since November 2025. team-solve was tuned on ten Verified issues; without them the scores are 77.8% and 80.2%. Each issue ran once. Fable 5 is about 95%.

In the first run, OpenRouter's web search reached upstream fixes on 126 issues. Those 126 ran again with the web tools removed. The [run's README](https://github.com/guregodevo/swe-bench#what-went-wrong-and-what-was-run-again) has the details.

[guregodevo/swe-bench](https://github.com/guregodevo/swe-bench) · [Results in Memdoor](https://github.com/guregodevo/memdoor/tree/main/examples/workflows/team-solve/bench/memdoor-swe-bench) · [team-solve](https://github.com/guregodevo/memdoor/tree/main/workflows/team-solve) · [Team mode](https://memdoor.ai/docs/team-mode) · [Memdoor](https://memdoor.ai)
