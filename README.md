# Argocd-diff-preview testcases

## Prerequisites

* Local cluster with argocd installed in namespace `argocd-diff-preview`
* Active kubeconfig pointing to the local cluster
* Binary `argocd-diff-preview` on path

## Commands

Installing argocd

```sh
helm upgrade argo-cd argo/argo-cd \
    --version 9.5.12 \
    --namespace argocd-diff-preview \
    --install \
    -f my-values.yaml
```

Checkouts

```sh
rm -rf base-branch
rm -rf target-branch

git clone https://github.com/TBeijen/argocd-diff-testcases base-branch --depth 1 -q
git clone https://github.com/TBeijen/argocd-diff-testcases target-branch --depth 1 -q -b pr-1
```

```sh
# Uses binary
argocd-diff-preview \
  --argocd-namespace=argocd-diff-preview \
  --create-cluster=false \
  --target-branch=pr-1 \
  --repo=TBeijen/argocd-diff-testcases \
  --render-method=repo-server-api
```
