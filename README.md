# gitops-deploy

Manifests for a [Beget](https://beget.com/en/p1835900) managed k8s cluster

[docs](https://beget.com/en/kb/manual/managed-kubernetes)

Source of truth for what's running

Two repos: push to [gitops-app](https://github.com/kirqe/gitops-app) `main` → CI
builds the image and commits the new tag here → Argo CD in the cluster pulls this
repo and reconciles (pull-based GitOps; CI never touches the cluster)

## GitHub setup

`beget-k8s` environment with two secrets:

- **KUBECONFIG** — `cat kubeconfig.yaml | base64 -w 0`
- **DEPLOY_TOKEN** — classic PAT, repo scope

Important parts:

- `ci.yml` must set `environment: beget-k8s` (else env-scoped secrets are
  invisible → "password auth not supported" push failure).
- Settings → Actions → General → Workflow permissions → **Read and write**.

## Bootstrap (once)

Ingress controller:

```bash
kubectl apply -k infrastructure/ingress-nginx/
kubectl -n ingress-nginx get svc ingress-nginx-controller -w   # wait for EXTERNAL-IP
```

Point app DNS A records at the LB IP; nginx routes by Host.

Argo CD + register the app:

```bash
kubectl create namespace argocd
kubectl apply -k infrastructure/argocd/
kubectl apply -f bootstrap/gitops-app.yaml      # one Application per app
```

From here Argo watches this repo and reconciles `apps/gitops-app/` (auto-sync,
self-heal, prune). `KUBECONFIG` is no longer used by CI.

## Deploy

Each app's CI bumps its image tag in `apps/<app>/deployment.yaml` and pushes.
Argo CD picks up the commit and rolls it out — no `kubectl apply` from CI.
