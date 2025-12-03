# Kubernetes GitOps Repository

## Current Structure

### Job 1: Initial ArgoCD Bootstrap (COMPLETED)

```
platform/
├── kustomization.yaml
└── argocd/
    ├── kustomization.yaml
    └── bootstrap.yaml
```

### Job 2: MetalLB Load Balancer (COMPLETED)

```
platform/
├── kustomization.yaml (updated)
├── argocd/
│   ├── kustomization.yaml
│   └── bootstrap.yaml
└── metallb/
    ├── kustomization.yaml
    └── base/
        ├── kustomization.yaml
        ├── metallb-application.yaml
        └── metallb-config.yaml
```

**What was created:**
- `platform/metallb/kustomization.yaml` - References base folder
- `platform/metallb/base/kustomization.yaml` - References metallb-application.yaml and metallb-config.yaml
- `platform/metallb/base/metallb-application.yaml` - ArgoCD Application for MetalLB native manifests
- `platform/metallb/base/metallb-config.yaml` - IPAddressPool (10.10.10.200-250) + L2Advertisement

**MetalLB Application Details:**
- **Name:** metallb
- **Repo:** https://github.com/metallb/metallb.git
- **Version:** v0.13.12
- **Install:** Native manifests (config/manifests) - works with Cilium kube-proxy replacement
- **Namespace:** metallb-system (auto-created)
- **Sync Policy:** Automated with prune + selfHeal
- **IP Range:** 10.10.10.200-10.10.10.250

**To Deploy:**
```bash
# Bootstrap will automatically deploy this via platform-root Application
kubectl -n argocd get application metallb -w
```

## Deployment Flow

1. Apply `platform/argocd/bootstrap.yaml` → platform-root Application syncs
2. platform-root syncs `platform/` → ArgoCD reads root kustomization.yaml
3. Root kustomization includes both `argocd/` and `metallb/`
4. MetalLB Application deploys native manifests from metallb repo
5. MetalLB config creates IPAddressPool and L2Advertisement

## Next Steps

Job 3 will add Gateway API and Cilium Gateway integration.
