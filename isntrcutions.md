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

**What was created:**
- `platform/kustomization.yaml` - References argocd folder
- `platform/argocd/kustomization.yaml` - References bootstrap.yaml
- `platform/argocd/bootstrap.yaml` - App-of-Apps root Application

**Bootstrap Application Details:**
- **Name:** platform-root
- **Namespace:** argocd
- **Repo:** https://github.com/Ealoltm/k8s-gitops.git
- **Branch:** main
- **Path:** platform
- **Sync Policy:** Automated with prune + selfHeal
- **Sync Options:** CreateNamespace=true

**To Deploy:**
```bash
kubectl apply -f platform/argocd/bootstrap.yaml
```

## Next Steps

Job 2 will add MetalLB and additional platform components (cert-manager, gateway-api, sealed-secrets, monitoring, image-updater, velero, and sample apps).
