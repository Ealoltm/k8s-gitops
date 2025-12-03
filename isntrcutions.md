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

### Job 3: Gateway API CRDs (COMPLETED)

```
platform/
├── kustomization.yaml (updated)
├── argocd/
│   ├── kustomization.yaml
│   └── bootstrap.yaml
├── metallb/
│   ├── kustomization.yaml
│   └── base/
│       ├── kustomization.yaml
│       ├── metallb-application.yaml
│       └── metallb-config.yaml
└── gateway-api/
    ├── kustomization.yaml
    └── base/
        ├── kustomization.yaml
        └── gateway-api-application.yaml
```

**What was created:**
- `platform/gateway-api/kustomization.yaml` - References base folder
- `platform/gateway-api/base/kustomization.yaml` - References gateway-api-application.yaml
- `platform/gateway-api/base/gateway-api-application.yaml` - ArgoCD Application for Gateway API CRDs v1.0.0

**Gateway API Application Details:**
- **Name:** gateway-api-crds
- **Repo:** https://github.com/kubernetes-sigs/gateway-api.git
- **Version:** v1.0.0
- **Install:** CRD manifests (config/crd/kubernetes)
- **Namespace:** kube-system
- **Sync Policy:** Automated with prune + selfHeal

**To Deploy:**
```bash
kubectl -n argocd get application gateway-api-crds -w
```

### Job 4: Cilium Gateway (COMPLETED)

```
platform/
├── kustomization.yaml (updated)
├── argocd/
│   ├── kustomization.yaml
│   └── bootstrap.yaml
├── metallb/
│   ├── kustomization.yaml
│   └── base/
│       ├── kustomization.yaml
│       ├── metallb-application.yaml
│       └── metallb-config.yaml
├── gateway-api/
│   ├── kustomization.yaml
│   └── base/
│       ├── kustomization.yaml
│       └── gateway-api-application.yaml
└── ingress/
    └── cilium-gateway/
        ├── kustomization.yaml
        ├── application.yaml
        └── base/
            ├── kustomization.yaml
            ├── gatewayclass.yaml
            └── gateway.yaml
```

**What was created:**
- `platform/ingress/cilium-gateway/kustomization.yaml` - References base folder
- `platform/ingress/cilium-gateway/base/kustomization.yaml` - References gatewayclass and gateway manifests
- `platform/ingress/cilium-gateway/base/gatewayclass.yaml` - GatewayClass with Cilium controller
- `platform/ingress/cilium-gateway/base/gateway.yaml` - Gateway on IP 10.10.10.201 (HTTP + HTTPS)
- `platform/ingress/cilium-gateway/application.yaml` - ArgoCD Application for Cilium Gateway deployment

**Cilium Gateway Details:**
- **GatewayClass:** cilium (controller: io.cilium/gateway-controller)
- **Gateway:** cilium-gateway in cilium-gateway namespace
- **Address:** 10.10.10.201
- **Listeners:** HTTP (port 80), HTTPS (port 443)
- **Sync Policy:** Automated with prune + selfHeal

**To Deploy:**
```bash
kubectl -n argocd get application cilium-gateway -w
```

## Next Steps

Job 5 will add cert-manager for certificate management.
