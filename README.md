# Argocd-diff-preview testcases

## Prerequisites

* Local cluster with argocd installed in namespace `argocd-diff-preview`
* Active kubeconfig pointing to the local cluster
* Binary `argocd-diff-preview` on path


## Commands

```sh
helm upgrade argo-cd argo/argo-cd \
    --version 9.5.12 \
    --namespace argocd-diff-preview \
    --install \
    -f my-values.yaml
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
