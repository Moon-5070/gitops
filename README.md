# GitOps for cluster-2

This repository contains Kubernetes manifests for deploying test workloads to cluster-2 with Argo CD.

Argo CD is installed on cluster-1, and cluster-2 is already registered as a deployment target.

## Application

- Application name: `cluster2-nginx`
- Target cluster API server: `https://192.168.160.15:6443`
- Target namespace: `nginx-test`
- Manifest path: `apps/cluster2-nginx`

## Repository Structure

```text
gitops/
  apps/
    cluster2-nginx/
      deployment.yaml
      service.yaml
      kustomization.yaml
  argocd-apps/
    cluster2-nginx-application.yaml
  README.md
```

## Create the Argo CD Application with CLI

```bash
argocd app create cluster2-nginx \
  --repo https://github.com/Moon-5070/gitops.git \
  --path apps/cluster2-nginx \
  --revision main \
  --dest-server https://192.168.160.15:6443 \
  --dest-namespace nginx-test \
  --sync-option CreateNamespace=true
```

## Sync the Application

```bash
argocd app sync cluster2-nginx
```

## Verify on cluster-2

```bash
KUBECONFIG=~/cluster2-config kubectl get all -n nginx-test
```
