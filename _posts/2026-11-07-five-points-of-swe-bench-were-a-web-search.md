---
layout: post
title: "Five points of my SWE-bench score came from a web search I thought was blocked"
date: 2026-11-07
---

> One door left open:<br>
> the answers walked in through it.<br>
> Eighty, honestly.

I ran all 500 instances of [SWE-bench Verified](https://www.swebench.com) with [Memdoor](https://memdoor.ai) on DeepSeek V4.1 Flash, two ways: one agent turn on the issue, and [team-solve](https://github.com/guregodevo/memdoor/tree/main/workflows/team-solve), a direct attempt checked with the project's own tests and, when the check fails, a team that tries again. The first full run said team-solve resolves 85.2%.

Then I read the traces. The honest number is 80.0%.

## The proxy allowed one host

Each instance ran in its own container, the repository at `/testbed`, a Memdoor gateway inside, and one way out: a proxy that opens `openrouter.ai:443` and refuses everything else. The model is on OpenRouter, so that host had to stay open. Nothing else did.

The agents tried. `web_fetch` was called 857 times, on `github.com`, `raw.githubusercontent.com`, `patch-diff.githubusercontent.com`, the Django tracker. The proxy refused every one.

`web_search` never touched the proxy's list. It is OpenRouter's own server-side search: the request goes to openrouter.ai, OpenRouter searches the web, the answer comes back from openrouter.ai. It answered 1,046 times, on 126 instances. On `astropy__astropy-13398`, which is upstream PR #13398:

> `itrs_observed_transforms.py` lives in `astropy/coordinates/builtin_frames/` … (introduced via issue #13355 / PR #13398). **Original `itrs_to_observed_mat(observed_frame)`**: … returns `minus_x @ rotation_matrix(PIOVER2 - elat, 'y') @ rotation_matrix(elong, 'z')`

That is the fix, handed to the agent writing it. In the team arm, on 58 instances an answer named the SWE-bench instance itself.

## Run them again without the web

The 126 instances ran again in both arms, the same binary, with `web_search` and `web_fetch` taken out of every agent's tools and a stop on any model request that still offered one. The first rerun stopped within minutes: the tools were removed by a watcher next to the workflow, and on `django__django-16938` the workflow read them a second before the watcher did. The agents searched; the run stopped; the attempt was thrown away. Now the machine signs in, the tools go, and only then does the workflow start. Over 252 rerun runs, no request offered a web tool.

On those 126 the team went from 116 resolved to 90, the single turn from 103 to 88. The other 374 never got a search answer and keep their first-run predictions.

## The score

| | resolved | per instance | median time |
|---|---|---|---|
| one agent turn | **388** / 500 (77.6%) | $0.012 | 91 s |
| team-solve | **400** / 500 (80.0%) | $0.022 | 258 s |
| team-solve's direct attempt | 385 / 500 (77.0%) | | |

SWE-bench's own tools re-derived both lists from every instance's test output (`swebench submit package`), and `swebench submit verify` passes. The 1,000 runs that count cost $16.85 on OpenRouter.

Against its own direct attempt, the team gained 15 and lost none: when the direct attempt passes the project's check it stands, and when the team does no better, `keep` puts it back. Against the single turn the team gained 27 and lost 15. That is +2.4 points, and p = 0.09: suggestive, not proven.

## The kit had bugs too

- Run 1 stopped after about 140 instances on a Docker layer that failed to extract, most likely two containers pulling one image while the scorer removed images. Nothing of it is used; run 2 started from the first instance.
- The stop that caught `django__django-16938` left two containers *exited*, not removed, and the restart took them for runs in flight. Neither had written a prediction; both ran again.
- The same stop cut the scorer between a single score and a team score. The scorer keys on the single score, so it never came back for the team's. Two instances were scored again.
- `pgrep -f kit/score2.sh` inside an ssh command matched the ssh command, which names the script. The scorer I thought I had restarted was never started.

The last three were found by a count that did not add up: 65 runs of 66, 25 team scores of 26.

## What it is not

Not a leaderboard entry: SWE-bench Verified takes submissions only from academic teams and research labs since November 2025. Both runs are in the leaderboard's layout anyway, so anyone can check them. Not a held-out set for team-solve: it was tuned on ten Verified instances; without them the arms score 77.8% and 80.2%, and on the ten both resolve 7. One run per instance, not a mean. And not close to the frontier: Fable 5 is about 95%.

What it is: a cheap open model at 80% for two cents an instance, every trace public, and a web search I would not have caught without reading them.

[The run: predictions, logs, traces, the kit](https://github.com/guregodevo/swe-bench) · [Results](https://github.com/guregodevo/memdoor/tree/main/examples/workflows/team-solve/bench/memdoor-swe-bench) · [team-solve](https://github.com/guregodevo/memdoor/tree/main/workflows/team-solve) · [Memdoor](https://memdoor.ai)
