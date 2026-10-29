---
layout: post
title: "A local model inside my GitHub CI job"
date: 2026-10-29
---

> No key in the job.<br>
> Nine billion weights on four cores<br>
> catch the loop that slipped.

Two days ago [Memdoor](https://memdoor.ai) became one step of a GitHub job: a review on every pull request, on your own model key. The first question I asked myself after shipping it was the one a stranger would ask: *do I have to hand a key to CI to try this?*

So I tried the other way. Instead of passing a key, set the model as local. No install, nothing on my machine, everything inside the GitHub Action:

```yaml
- uses: guregodevo/memdoor-action@v1
  with:
    workflow: pr-review
    local-model: qwen3.5:9b
```

The action starts [Ollama](https://ollama.com) on the runner, downloads and caches the model, and points Memdoor at it. No secret, no API account, and nothing leaves the runner.

## The first run failed

A free GitHub runner is 4 CPUs and 16 GB of memory. A 9B model is about the largest that fits, and on those cores it reads a prompt at about 12 tokens a second.

The review I ship on a key is an agent: it reads the diff, opens the files around it, runs the code to check each finding. Its first prompt alone is 5,600 tokens. On the runner that is eight minutes of silence before the first word, and Memdoor's watchdog, which treats a provider that says nothing for 60 seconds as dead, killed it twice. The 4B model was worse: it *thought* for 1,300 tokens at 2.6 a second and hit the step's timeout.

Three changes made it work:

- **One model call, not an agent.** On a local model the review reads the diff and writes the findings in a single call, with thinking off. No tools, nothing run.
- **A runtime on your own machine may be slow.** A keyless provider (Ollama, llama.cpp) gets ten minutes of silence before the watchdog gives up, not sixty seconds. A remote API that goes quiet for a minute is still dead.
- **A cut-off answer is not a review.** The 4B wrote for ten minutes, the clock cut it, and the half-written text passed as the review. Now an answer stopped by its clock fails the step.

^

Then the 9B caught a planted off-by-one as a must-not-merge finding in 54 seconds.

## Then I measured it against a key

A demo bug proves the plumbing, not the reviewer. So I opened the same three pull requests on [llamafit](https://github.com/guregodevo/llamafit), a small Go project of mine, and reviewed each one twice: once on the model inside the runner, once on my OpenRouter key.

| Pull request | No key: `qwen3.5:9b` in the runner | My key: OpenRouter, Memdoor's default model |
|---|---|---|
| An obvious bug: a loop that now skips the last layer | P0 on the right line, but explained by a panic that cannot happen | P0: the last layer is left out of every estimate |
| A subtle bug: the returned path built from the lowercased file name | missed, the job passed | P0: `MODEL.GGUF` comes back as a path that does not exist |
| The same change done correctly | passed, no false P0 | passed, no false P0 |
| Review | 66–78 s | 48–60 s |
| Job | 3½–5 min, most of it the model's first download | about 1 min |

The subtle one is a pull request that accepts `.GGUF` files in any case, and in passing builds the returned path from the *lowercased* name. On Linux, `MODEL.GGUF` comes back as `model.gguf`, a file that does not exist. The local model let it through. In an earlier run, asked to show its reasoning, it had read that very line and decided it preserved the original filename. The agent on a key flagged it as a P0 in 48 seconds.

The obvious one is worth a second look too. The local model put its P0 on the right line, and explained it with a panic that cannot happen. It was right that something was wrong, and wrong about what.

## The honest label

So the feature shipped under the name it earned: **a free first pass, not a reviewer.** It points at the obvious and stays quiet on correct code. That is worth something on a public repository where you would rather not put a secret, or on a team that has no API account yet. For a review that has to be right, the model needs to open the files and run the code, and on a free runner that means a key.

What I like is that the job does not care which one you pick. Either way it passes or fails on a check, never because the model said so.

[Docs: In CI](https://memdoor.ai/docs/ci) · [The action](https://github.com/marketplace/actions/memdoor) · [GitHub](https://github.com/guregodevo/memdoor)
