---
layout: post
title: "CI already knew"
date: 2026-10-27
---

> Trust no agent's word.<br>
> The exit code says it's done.<br>
> Red means it is not.

![A workflow drawn as a river: the streams diff and tests meet in the pool of review, pass under the gate of verdict, and reach the sea, exit 0.](/assets/images/ci-already-knew.svg)

CI never believed anyone. It believes an exit code. So now [Memdoor](https://memdoor.ai) runs as one step of a GitHub job, on your own key:

```yaml
- uses: guregodevo/memdoor@v1
  with:
    workflow: pr-review
    openrouter-api-key: ${{ secrets.OPEN_ROUTER_API_KEY }}
```

- **pr-review** runs the code, and fails the job only on a real blocker.
- **fix-tests** is done when the tests pass, not when the agent says so.
- **release-notes** turns the commits since the last tag into plain words.

The model writes. The exit code decides.

[Memdoor](https://memdoor.ai) · [GitHub](https://github.com/guregodevo/memdoor) · [Marketplace](https://github.com/marketplace/actions/memdoor-workflow)
