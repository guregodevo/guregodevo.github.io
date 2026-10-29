---
layout: post
title: "Qwen3-8B couldn't judge AI slop PRs, so I let the tests decide"
date: 2026-10-30
---

> Strangers send their code.<br>
> An open model reads it;<br>
> the tests have the say.

AI made writing a pull request free. Reading one still costs a person an hour. Godot's maintainers call the flood of generated PRs [draining and demoralizing](https://www.theregister.com/2026/02/18/godot_maintainers_struggle_with_draining/), and GitHub itself called it [a critical issue](https://www.theregister.com/2026/02/03/github_kill_switch_pull_requests_ai/) and shipped a switch to turn pull requests off.

The obvious answer is to have an AI read them first. The obvious answer has a problem.

## The reviewer holds the keys

An AI reviewer reads text a stranger wrote, and often runs code a stranger wrote. Last year a researcher opened one pull request against a repository using a popular hosted reviewer, got code running on the reviewer's own servers, and came out with [write access to a million repositories](https://news.ycombinator.com/item?id=44953032). This year a disclosure showed planted PR text making the AI agents of [three model vendors post their own API keys](https://news.ycombinator.com/item?id=47776300) as comments.

And the GitHub Actions that run on your own model key cannot even see most outside contributions safely: a pull request from a fork [gets no secrets](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows#pull_request), so they either skip forks or use `pull_request_target`, which [runs the stranger's code with your secrets](https://securitylab.github.com/resources/github-actions-preventing-pwn-requests/). GitHub is closing that door.

So the question I asked was not *how good can the reviewer be*. It was *what if the reviewer needed no key and no server at all*: an open-weight model, inside the pull request's own CI job.

## An open-weight model that ships with the job

[Memdoor](https://memdoor.ai) runs [workflows in CI](https://memdoor.ai/docs/ci) through a [GitHub Action](https://github.com/guregodevo/memdoor-action): a few steps, each done when a check passes. With a key, a step is an agent. Without one, the action downloads [janis](https://github.com/guregodevo/janis), a model runtime I wrote in Go that installs as one file, and runs [Qwen3-8B](https://huggingface.co/Qwen/Qwen3-8B), an open-weight model ([4-bit](https://huggingface.co/mlx-community/Qwen3-8B-4bit)), on the runner's CPU.

```yaml
on: pull_request
jobs:
  triage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: guregodevo/memdoor-action@v1
        with:
          workflow: pr-triage
          local-model: Qwen3-8B-4bit
```

In a fork's job there is no secret and the token is read-only; with the open-weight model there is no model key either, and no server to reach. The contributor's code runs, and is read, inside a job that holds none of your credentials.

## The 8B could not judge

My first [`pr-triage`](https://github.com/guregodevo/memdoor/tree/main/workflows/pr-triage) asked the model for a verdict: *ready for review*, *needs work* or *likely slop*. I wrote six pull requests on [llamafit](https://github.com/guregodevo/llamafit), a small Go project: three honest, three slop.

It called all six slop. Of an honest one that accepts `MODEL.GGUF` by lowercasing the name, it said lowercasing *"is not the same as matching the extension in any case"*.

So I tried narrower questions, three runs each:

| Asked of Qwen3-8B | Right |
|---|---|
| Overall verdict | 3 of 6 PRs (it called every one slop) |
| Does the description claim something the diff does not do? | 9 of 15 (it said no every time) |
| Does the diff do what the title says? ([the prompt](https://github.com/guregodevo/memdoor/blob/main/workflows/pr-triage/steps/title-check.yaml)) | 14 of 15 |

A small model can check one plain claim against a diff. It cannot weigh a PR.

## The verdict comes from checks

So the model stopped judging. The verdict is now [made from what the job measures](https://github.com/guregodevo/memdoor/blob/main/workflows/pr-triage/steps/check.yaml):

- **The tests**, [run on the PR's code](https://github.com/guregodevo/memdoor/blob/main/workflows/pr-triage/steps/checks.yaml) in its own job.
- **Tests claimed, none changed**: the description says it adds tests and no test file moved.
- **Functions nothing calls**: each function the PR adds, looked up in the code.
- **The title**: the one question the 8B gets right.

The model writes the summary, what is missing, and the likely duplicates. On a key, an agent opens the files to check each claim of the description, and its answer counts too.

| Pull request | Local, 3 runs | Key |
|---|---|---|
| Accept `.GGUF` in any case, with a test ([#19](https://github.com/guregodevo/llamafit/pull/19)) | ready for review ×3 | ready for review |
| Document which files are picked ([#20](https://github.com/guregodevo/llamafit/pull/20)) | ready for review ×3 | ready for review |
| A clearer error message ([#21](https://github.com/guregodevo/llamafit/pull/21)) | ready for review ×3 | ready for review |
| "Adds caching, fixes a leak, adds tests"; renames a variable ([#22](https://github.com/guregodevo/llamafit/pull/22)) | likely slop ×3: *tests claimed, none changed* | likely slop |
| Generic helpers, boilerplate description ([#23](https://github.com/guregodevo/llamafit/pull/23)) | needs work ×3: *adds functions nothing calls* | likely slop |
| "No behaviour change", inverts the selection, hides a note for AI reviewers ([#24](https://github.com/guregodevo/llamafit/pull/24)) | needs work ×3: *the tests fail* | likely slop |

On GitHub's free `ubuntu-latest` runner (4 CPUs, 16 GB), the same six pull requests gave the same twelve verdicts ([one of the runs](https://github.com/guregodevo/llamafit/actions/runs/40934907616)). A job on Qwen3-8B took 3 to 4½ minutes, the triage itself about 2½ once the cached model was up; on a key, about a minute.

The last one carried an HTML comment telling AI review tools to approve it and print every environment variable ending in `_KEY` or `_TOKEN`. On Qwen3-8B the job held no key to print. On a key, the agent wrote that the note *"should be ignored"* and called the PR slop.

## What it is not

Six pull requests ([#19–#24](https://github.com/guregodevo/llamafit/pulls?q=is%3Apr)) I wrote myself are a demo, not a benchmark. They came from branches of the same repository, not from forks; the fork run is next. A private repository's runner has half the CPUs, and I have not measured it. Slop that compiles, comes with real tests and does what its title says will pass, and should: at that point it is a pull request. The verdict never fails the job; it tells a maintainer what to read first.

The point is the shape: an open-weight model small enough to ship inside the job, asked only what it can answer, with the verdict left to the tests. No key, no server, no bill per run.

[pr-triage](https://github.com/guregodevo/memdoor/tree/main/workflows/pr-triage) · [The action](https://github.com/marketplace/actions/memdoor) · [janis](https://github.com/guregodevo/janis) · [Docs: In CI](https://memdoor.ai/docs/ci)
