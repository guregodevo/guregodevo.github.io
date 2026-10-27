---
layout: post
title: "CI already knew"
date: 2026-10-27
---

In [the last post](/2026/10/06/workflows-not-skills/) I argued that an agent's work should be a graph of steps, each one done when its *target exists* — a file is there, a command exits 0 — and never because the model says so. Writing it, I kept thinking that none of this is new to anyone who runs CI. A CI job has never believed a word anyone said. It believes an exit code.

So the natural place for a [Memdoor](https://github.com/guregodevo/memdoor) workflow is one step of a GitHub job. It is now [on the Marketplace](https://github.com/marketplace/actions/memdoor-workflow):

```yaml
on: pull_request
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: guregodevo/memdoor@v1
        with:
          workflow: pr-review
          openrouter-api-key: ${{ secrets.OPEN_ROUTER_API_KEY }}
```

That is the whole setup. The runner installs one binary, starts a gateway on localhost, runs the workflow on your key, and the job ends with the run's outcome. Nothing is added to the repository.

## A review that has to run the code

`pr-review` is three steps. The first is a shell command: it takes the pull request against its base branch, from the shallow checkout GitHub hands it, and writes the diff to a file. The second is the agent: it reads the diff, opens the files around it, and writes a review where every finding has a file, a line and a severity — **P0** for what must not merge, P1 for what should be fixed, P2 for a nit. The third is a shell command again. It reads the review and exits 1 if there is a P0.

The model never decides whether the job passes. It writes a file; a ten-line script reads the file; the exit code fails the job. The review is in the job summary either way.

I tried it on a pull request that added a `Mean` function and, on the way, "tidied" a loop in `Sum` from `i < len(xs)` to `i < len(xs)-1`. The review came back with one P0 on that line, and the reason was not an opinion: it had run `Sum([]int{1,2,3})` and got 3. It also flagged, as a P1, that `Mean` divides by zero on an empty slice. The step exited 2, which is a red job. On a pull request that added a correct `Max` with its test, it ran `go vet` and `go test`, wrote "No findings", and exited 0.

That last part matters as much as the first. A reviewer that complains about everything gets switched off in a week. This one only stops a merge for the thing it proved.

## A fix whose definition of done is the tests

`fix-tests` finds the project's test command — `go test ./...` from a `go.mod`, `cargo test`, `npm test`, pytest — and gives the coder one step whose target is that command passing. Not "the agent says the tests pass": the engine runs them.

Two consequences fall out of that, and neither needed writing. When the tests already pass, the target already holds, so the step is skipped and no model is called at all; on a green build the workflow costs nothing. When they fail, the run is not done until they pass, whatever the agent concludes. On a `Reverse` function that dropped the first rune, the coder changed `i > 0` to `i >= 0` in eighteen seconds and left the tests alone. The fix stays in the working tree with a report of the cause and the diff; whether it becomes a commit or a pull request is the next step of your job, not the agent's.

The third built-in, `release-notes`, turns the commits since the last tag into what a person notices. On my test repository it kept the new `--json` flag and the crash fix, and left the internal rename out.

## The gate, honestly

A workflow can wait for a person: an external task nobody runs, opened with an approval. On a laptop or a box that stays on, that is the point of it. In a CI job there is nobody to open it, and the polite failure would be a job that sits there until GitHub kills it six hours later. So in CI a run that reaches a gate stops with exit code 3 and prints what the approver would have read. Exit 0 is done, 2 is failed, 1 is stopped, 3 is "this needed a person". Four outcomes, each one checkable.

## Your key, and where it is

The action runs on your model key, which means the agent runs the repository's code on a machine where your key is. The boundary that matters is GitHub's: a pull request from a fork gets no secrets, so a stranger's code never runs beside your key. Trigger on `pull_request`, never `pull_request_target`, which runs their code with your secrets. Memdoor adds a second line — the commands the agent runs no longer see model keys in their environment, so an `env` or a test that dumps it cannot carry one into the model's context or the job log — but it is a second line, and I would rather say so than oversell it.

## The same files everywhere

The part I like best is that none of this is CI-specific. `pr-review` is a directory of three YAML files. The same directory runs in the terminal window, from `memdoor workflow run pr-review` in a script, or on a schedule with `memdoor cron add`, on whatever gateway you own. Write your own and it is `workflow: <name>`, from `.memdoor/workflows/<name>/` in the repository.

CI was the first place in software that refused to take anyone's word for it. It seems right that agents should end up there too, held to the same standard as everything else in the pipeline: an exit code.

Memdoor is open source: [github.com/guregodevo/memdoor](https://github.com/guregodevo/memdoor). The action is on the [GitHub Marketplace](https://github.com/marketplace/actions/memdoor-workflow).
