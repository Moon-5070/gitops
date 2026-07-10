# Multi-cluster GitOps

This repository is a multi-cluster GitOps experiment for Argo CD.

Argo CD runs on cluster1 as the hub/management cluster. cluster2 is registered in Argo CD as a target cluster.

## Applications

### common-echo

`common-echo` is deployed to every cluster managed by Argo CD, including cluster1 and cluster2.

It uses an ApplicationSet Cluster generator with `clusters: {}`.

### target-echo

`target-echo` is deployed only to clusters that have the `env=target` label on their Argo CD cluster Secret.

In this setup, that label should be applied to the cluster2 Secret, so `target-echo` is deployed only to cluster2.

## Repository Structure

```text
gitops/
  apps/
    common-echo/
      deployment.yaml
      service.yaml
      kustomization.yaml
    target-echo/
      deployment.yaml
      service.yaml
      kustomization.yaml
  argocd-apps/
    common-echo-all-applicationset.yaml
    target-echo-cluster2-applicationset.yaml
  README.md
```

## Label cluster2 for target-only deployment

List Argo CD cluster Secrets:

```bash
kubectl get secret -n argocd -l argocd.argoproj.io/secret-type=cluster
```

Apply the `env=target` label to the cluster2 Secret:

```bash
kubectl label secret -n argocd <cluster2-secret-name> env=target --overwrite
```

## Apply ApplicationSets

```bash
kubectl apply -f argocd-apps/common-echo-all-applicationset.yaml
```

```bash
kubectl apply -f argocd-apps/target-echo-cluster2-applicationset.yaml
```

## Check Argo CD Applications

```bash
argocd app list
```

## Verify cluster1

```bash
kubectl get all -n common-test
```

## Verify cluster2

```bash
KUBECONFIG=~/cluster2-config kubectl get all -n common-test
KUBECONFIG=~/cluster2-config kubectl get all -n echo-test
```

## Test service access inside cluster2

```bash
KUBECONFIG=~/cluster2-config kubectl run curl-common-test --image=curlimages/curl -it --rm --restart=Never -- curl common-echo.common-test.svc.cluster.local
```

```bash
KUBECONFIG=~/cluster2-config kubectl run curl-target-test --image=curlimages/curl -it --rm --restart=Never -- curl target-echo.echo-test.svc.cluster.local
```
