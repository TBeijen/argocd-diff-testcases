# Transient chart dependency build errors: Original error swallowed + retry mechanism

## Description

When running ~250 applications through the repo-server-api render method, we quite frequently hit chart download errors, making the entire CI run fail. These errors are not consistent, not limited to specific chart repos, so hard to reproduce.

Two things at play:

* The transient errors are returned as a permissionDenied error. It looks like the original error is replaced because no ProjectRepos are sent to repo-server. (See analysis below, I'll create a PR for this)
* Might be worth considering a retry mechanism, possibly controled by cli arg. (If not already present, haven't investigated yet)

### PermissionError

This is triggered by logic in argocd itself, but can probably be avoided by a small change in this project. See: https://github.com/argoproj/argo-cd/blob/master/reposerver/repository/repository.go#L1363

Sequence:

1. `helm template` fails with a missing dependency error
2. `runHelmBuild()` is called to fetch dependencies via `helm dependency build`
3. `helm dep build` fails transiently (CDN timeout: "cannot be reached" or "could not download")
4. Repo-server iterates Helm repos, checking each against `ProjectSourceRepos` via `isSourcePermitted()`
5. Since argocd-diff-preview sends empty `ProjectSourceRepos`, every repo fails the permission check
6. Because of that, `PermissionDenied` error is returned: `"helm repos X are not permitted in project ''"`
7. This error is returned _instead of_ the original error

Fix is probably as simple as sending `*` as project sourceRepos.

To reproduce, see: https://github.com/TBeijen/argocd-diff-testcases

`examples/local-chart-with-deps/app.yaml` contains a subchart dependency with an incorrect url. This results in a seemingly unrelated permission error:

```
❌ failed to render app local-chart-with-deps [t|examples/local-chart-with-deps/app.yaml]: repo server returned error for content source 0: failed to receive manifest response: rpc error: code = PermissionDenied desc = helm repos https://kubernetes.github.oopsie/ingress-nginx are not permitted in project ''
🕵️ Run with '--debug' for more details
```

### Retry mechanism

(not investigated yet, will do after first surfacing the original error)