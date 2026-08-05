# Fleet — GitOps deployment repository

This repo ships the **desired state** for our microservices in production. Each service repo only contains its source
code and a CI workflow; values and ArgoCD apps live here in fleet.

## Layout

```
fleet/
├── charts/
│   └── microservice/        # Generic reusable Helm chart
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           ├── _helpers.tpl
│           ├── deployment.yaml
│           └── service.yaml
├── env/
│   └── prod/                # Per-service values for prod only
│       ├── auth-values.yaml
│       ├── order-values.yaml
│       └── payment-values.yaml
└── argocd/
    ├── auth-service-prod.yaml
    ├── order-service-prod.yaml
    └── payment-service-prod.yaml
```

## How a deploy happens

1. CI in a service repo (e.g. `auth-service`) builds and pushes an image tagged with the short SHA.
2. CI then opens/edits `env/prod/<service>-values.yaml` in this fleet repo, bumping `image.tag`.
3. ArgoCD detects the git change, renders the chart, and syncs the cluster.
4. `kubectl` drift is auto-reverted by `selfHeal: true`.