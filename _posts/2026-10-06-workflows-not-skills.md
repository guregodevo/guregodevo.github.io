---
layout: post
title: "Workflows, not skills"
date: 2026-10-06
---

I started [Memdoor](https://memdoor.ai) in June with a small ambition: a lean coding agent in the terminal, written in Go, one binary, running on whatever key I already paid for. I had used the big agents and liked them. What I did not like was sitting next to them.

A coding agent is a loop: read, think, call a tool, read the result, think again. For a one-line fix the loop is wonderful. For the work that actually fills a day — vet and test, write the notes, wait for a review, tag, deploy, check — it is a loop I have to babysit. Each step is a prompt I type, an answer I read, a decision whether it really happened. The agent is fast; I am the bottleneck, and I am the only thing checking.

## Skills

The first answer the industry gave to this was the *skill*: a procedure written in prose, loaded into the model's context when its name is called. "To release: run vet and the tests, write the release notes, wait for approval, tag, deploy." Every agent has them now, under one name or another. I built them into Memdoor too, early, and they did what they do.

A skill is a recipe. The cook reads it and then cooks. Whether the steps happen, in which order, whether one was skipped because the model decided it was not needed today, is up to the cook. Run the same skill twice and you get two different runs. When the model says "done", it is done, because the only record of the run is the transcript, and the transcript is the model's account of itself.

I noticed the shape of my frustration before I could name it. The skill was never wrong. It was simply not the kind of thing that could be right. It describes work; it does not *structure* it.

## What I had already built

Years ago, for data pipelines, I had written a small DAG engine in Go, [mario](https://github.com/guregodevo/mario), in the spirit of Luigi. Its one idea, taken from Luigi, is that a task is done when its *target exists*: a file is there, a table has rows, a command exits 0. Readiness is derived from that. A task may start when every upstream is done or its target exists. Nothing is done because someone said so.

When I put the agent's work next to that engine, the mismatch with skills became obvious. A pipeline had every property my release skill lacked:

- **Done is a fact.** The target exists or it does not. The model's word does not count.
- **Dependencies are explicit**, so independent steps run in parallel and the rest wait exactly as long as they must.
- **A run is a record.** Each step keeps what it changed and whether it passed. A step that fails is fixed and the run resumes there; the five finished steps are not redone.
- **A person is a node.** An approval is a task only I can complete. The run builds everything up to it, stops, and shows me the diff.

So Memdoor's workflows are mario's workflows. You describe the steps and the agent writes them as files, one per task, in the repository:

```yaml
# .memdoor/workflows/ship/steps/tests.yaml
type: command
command: |
  go test -count=1 ./gateway/... ./pkg/... ./tools/... ./cmd/...
requires:
  - table_pattern: clean
```

```yaml
# .memdoor/workflows/ship/steps/deploy.yaml
type: command
command: |
  make deploy
requires:
  - table_pattern: push
  - table_pattern: approve
    external: true
```

The `external: true` is the gate. Nobody runs `approve`; the run waits there until I do. A task of `type: agent` is a turn of the coder with a prompt and a target — the review step of my own repository is one, and its target is the review file it must write. If the file is not there, the step did not happen, whatever the transcript says.

## Why this is the right unit

The argument is the same one data engineering settled fifteen years ago. Cron existed. Make existed. Shell scripts existed. Teams still moved to Luigi and Airflow, and not for the DAG picture. They moved for the run state, the retries, the idea that a task has a target, the fact that a failed pipeline on Tuesday is resumed on Wednesday rather than rerun. A recipe tells you what to do; an orchestrator holds you to it.

Agents are at the cron-and-make stage. A skill is a shell script the cook may or may not follow. The step that matters, the one that changes what you can trust, is to make "done" something the machine checks.

There is a second reason, and it is about cost. Every step that is a target the engine can check is a step the model does not have to narrate, re-read, or be asked about. The expensive part of an agent run is the context it drags along; a workflow throws most of it away between steps, because a step only needs its inputs and its target.

## Where skills still belong

Skills are not wrong; they are the wrong unit for the shape of a day's work. Inside a single step, a skill is exactly right: *how* to write a review, *how* to name a commit, the house rules of this repository. Memdoor's review workflow loads the `review` skill in its first step. The skill says how; the workflow says what, in which order, with what proof, and where I come in.

That is the whole position. Describe a procedure in prose and you get a procedure the model might follow. Describe it as a graph with targets and you get a run that either happened or did not, that you can resume, and that stops to ask you at the one place where your judgment is the point.

Memdoor is open source: [github.com/guregodevo/memdoor-oss](https://github.com/guregodevo/memdoor-oss). The engine is [mario](https://github.com/guregodevo/mario). The recording of the release workflow, gate and all, is on [memdoor.ai](https://memdoor.ai).
