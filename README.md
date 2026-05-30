# gitops-deploy

Manifests for a Beget-managed k8s cluster. Source of truth for what's running

## Bootstrap (once)

```bash
kubectl apply -k infrastructure/ingress-nginx/
kubectl -n ingress-nginx get svc ingress-nginx-controller -w   # wait for EXTERNAL-IP
```

Point app DNS A records at the LB IP; nginx routes by Host.

## Deploy

Each app's CI bumps its image tag in `apps/<app>/deployment.yaml`, pushes, then
`kubectl apply -R -f apps/`.
