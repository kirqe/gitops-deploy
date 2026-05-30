# gitops-deploy

Manifests for a [Beget](https://beget.com/en/p1835900) managed k8s cluster

[docs](https://beget.com/en/kb/manual/managed-kubernetes)

Source of truth for what's running

Two repos: push to [gitops-app](https://github.com/kirqe/gitops-app) `main` → CI
builds the image and commits the new tag here → CI applies the manifests to k8s

## GitHub setup

`beget-k8s` environment with two secrets:

- **KUBECONFIG** — `cat kubeconfig.yaml | base64 -w 0`
- **DEPLOY_TOKEN** — classic PAT, repo scope

Important parts:

- `ci.yml` must set `environment: beget-k8s` (else env-scoped secrets are
  invisible → "password auth not supported" push failure).
- Settings → Actions → General → Workflow permissions → **Read and write**.

## Bootstrap (once)

```bash
kubectl apply -k infrastructure/ingress-nginx/
kubectl -n ingress-nginx get svc ingress-nginx-controller -w   # wait for EXTERNAL-IP
```

Point app DNS A records at the LB IP; nginx routes by Host.

## Deploy

Each app's CI bumps its image tag in `apps/<app>/deployment.yaml`, pushes, then
`kubectl apply -R -f apps/`.
