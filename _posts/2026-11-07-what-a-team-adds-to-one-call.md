---
layout: post
title: "What a team adds to one call: 400 of 500 SWE-bench Verified issues at two cents each"
date: 2026-11-07
---

> One try, then a check.<br>
> When it fails, the team steps in.<br>
> Nothing it touched broke.

[team-solve](https://github.com/guregodevo/memdoor/tree/main/workflows/team-solve) is the harness behind [Memdoor](https://memdoor.ai)'s team mode. I ran it on all 500 issues of [SWE-bench Verified](https://www.swebench.com) against one agent turn on the same model, DeepSeek V4.1 Flash:

| | resolved | per issue | median time |
|---|---|---|---|
| one agent turn | 388 / 500 (77.6%) | $0.012 | 91 s |
| team-solve | **400** / 500 (80.0%) | $0.022 | 258 s |

SWE-bench's own tools re-derived both scores from each issue's test output, and every prediction, test log and trace is [public](https://github.com/guregodevo/swe-bench).

## The harness

The run starts the way one agent turn would. `direct` works on the issue as written while `plan` settles a Check, one command that exits 0 when the work is right, usually the project's own tests. `test-0` runs it on the direct attempt.

If the check passes, that attempt stands. If it fails, the team takes over from it: `context` gathers the files, two ideators propose an approach each, `solve` corrects the attempt, `test` and `review` check it, `repair` fixes what the review names, and a `judge` that saw none of the team's reasoning decides what a command cannot. Last, `keep` compares the team's result with the direct attempt and puts the direct attempt back when the team did not do better.

The planner, the ideators, the reviewer and the judge are single model calls with no tools. Only `direct`, `solve` and `repair` touch the tree. Every role starts from an empty context and reads only files on disk.

## What the team adds

`direct` is one agent turn, and it scored about what one agent turn does: 385, against 388. Everything team-solve gains is the team's.

| team-solve against | gained | lost | |
|---|---|---|---|
| its own direct attempt | 15 | **0** | p = 0.00006 |
| a separate single turn | 27 | 15 | p = 0.09 |

The first row is the design working. The team rewrote the direct attempt's patch on 63 issues; 15 of those went from failing to passing and none went the other way: `keep` puts back a direct attempt that passed its check, and one the team could not beat. The second row is +2.4 points against an independent run: suggestive, not proven by 500 issues.

The net gain is all on Django, 195 resolved against 183: a large project with a good test suite, where the Check has something to say. On sphinx the team does worse than the single turn, 30 against 33.

## What it costs

Twice the single turn: $10.84 for the 500, against $6.01. The team only runs where the check fails, so the median issue takes 258 s instead of 91 s and the slowest take an hour.

## How it was measured

Each issue ran in its own container from its SWE-bench image, with only the issue text going in, and one way out: a proxy that allows openrouter.ai and nothing else. That left one gap. `web_search` is OpenRouter's own search, answered through openrouter.ai, and in the first run it returned upstream fixes on 126 issues. Those 126 ran again in both arms with the web tools removed; the team's score went from 85.2% to the 80.0% above. The [run's README](https://github.com/guregodevo/swe-bench#what-went-wrong-and-what-was-run-again) has the details.

## Run it

In the Memdoor window, Shift+Tab to team mode and type the task, or `/w team-solve <the task>`. From a shell:

```
memdoor workflow run team-solve "Fix the failing date parser test; go test ./... must pass"
```

It runs in the project you are in, on whatever model you pick.

## What it is not

Not a leaderboard entry: SWE-bench Verified accepts submissions only from academic teams and research labs since November 2025. Not a held-out set: team-solve was tuned on ten Verified issues; without them the arms score 77.8% and 80.2%. One run per issue. And not the frontier: Fable 5 is about 95%. What it is: an open model at 80% for two cents an issue, and a team that never made its own first answer worse.

[The run: predictions, logs, traces, the kit](https://github.com/guregodevo/swe-bench) · [Results](https://github.com/guregodevo/memdoor/tree/main/examples/workflows/team-solve/bench/memdoor-swe-bench) · [team-solve](https://github.com/guregodevo/memdoor/tree/main/workflows/team-solve) · [Team mode](https://memdoor.ai/docs/team-mode)
