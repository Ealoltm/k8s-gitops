# Kubernetes GitOps Homelab - Platform Skeleton

This repository contains a complete GitOps platform skeleton for a Kubernetes homelab using **ArgoCD**, **Cilium**, **Gateway API**, **MetalLB**, and other production tools.

## Stack

- **Container Orchestration:** Kubernetes
- **GitOps Controller:** ArgoCD
- **CNI:** Cilium (already installed)
- **Ingress:** Gateway API (Cilium Gateway Controller)
- **Load Balancer:** MetalLB
- **Certificate Management:** cert-manager (controller only, no issuers)
- **Secrets:** Sealed Secrets
- **Monitoring:** kube-prometheus-stack
- **Image Updates:** ArgoCD Image Updater
- **Backups:** Velero
- **Sample Apps:** whoami, homepage

## Repository Structure

```
k8s-gitops/
├── clusters/
│   ├── home-lab/
│   │   ├── apps.yaml              # Platform + Apps Applications
│   │   └── kustomization.yaml
│   ├── dev/
│   ├── stage/
│   └── prod/
└── platform/
    ├── argocd/                     # ArgoCD self-managed
    ├── gateway-api/                # Gateway API CRDs
    ├── ingress/cilium-gateway/     # Cilium GatewayClass + Gateway
    ├── metallb/                    # Load Balancer config
    ├── cert-manager/               # Cert-manager controller
    ├── sealed-secrets/             # Secret encryption
    ├── monitoring/                 # Prometheus + Grafana
    ├── image-updater/              # ArgoCD Image Updater
    ├── velero/                     # Backup & restore
    └── kustomization.yaml          # Platform root
└── apps/
    ├── whoami/                     # Sample app with HTTPRoute
    └── homepage/                   # Sample app with HTTPRoute
```

## Bootstrap Process

1. **ArgoCD Bootstrap**: Apply `platform/argocd/bootstrap.yaml` to bootstrap the cluster
2. **Platform Deployment**: ArgoCD deploys all platform components
3. **Apps Deployment**: Sample applications deployed via HTTPRoutes

## Key Features

✅ **Fully GitOps-Managed**: Everything defined in Git, deployed by ArgoCD  
✅ **Gateway API v1**: Modern Kubernetes Ingress using Gateway API  
✅ **No Automatic TLS**: Manual cert-manager practice (Issuers, Certificates, CertificateRequests)  
✅ **No Domains**: Practice creating HTTPRoutes without hostnames  
✅ **Automated Sync**: All Applications with `automated: true` and `prune: true`  
✅ **CreateNamespace**: All namespaces created automatically  

## Platform Components

### Gateway API
- **CRDs**: Installed from `kubernetes-sigs/gateway-api`
- **GatewayClass**: Cilium controller (`io.cilium/gateway-controller`)
- **Gateway**: Cilium Gateway on ports 80 (HTTP) + 443 (HTTPS)

### cert-manager
- **Controller Only**: No ClusterIssuer configured
- **Manual Practice**: You create Issuers, Certificates, and CertificateRequests

### Applications
- **whoami**: Deployment + Service + HTTPRoute (path: `/whoami`)
- **homepage**: Deployment + Service + HTTPRoute (path: `/`)

## Getting Started

### 1. Bootstrap ArgoCD

```bash
# Apply bootstrap manifest
kubectl apply -f platform/argocd/bootstrap.yaml
```

### 2. Watch Sync

```bash
# Monitor Applications
kubectl -n argocd get applications -w

# Check platform components
kubectl get pods -A
```

### 3. Manual TLS Practice

Once apps are running:

```bash
# Create Issuer (self-signed example)
kubectl apply -f - <<EOF
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: selfsigned
  namespace: whoami
spec:
  selfSigned: {}
EOF

# Create Certificate
kubectl apply -f - <<EOF
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: whoami-tls
  namespace: whoami
spec:
  secretName: whoami-tls
  commonName: whoami.local
  issuerRef:
    name: selfsigned
    kind: Issuer
EOF

# Update HTTPRoute with TLS
# (Add hostname and TLS section to apps/whoami/httproute.yaml)
```

## Important Notes

- **No public domain**: All services accessed via Cilium Gateway IP
- **No automatic certificates**: Practice manual cert-manager workflows
- **Cilium only**: No kube-proxy (replaced by Cilium)
- **All namespaces auto-created**: By ArgoCD CreateNamespace sync option

## Repository URL

```
https://github.com/Ealoltm/k8s-gitops.git
```

## Next Steps

1. Review `platform/` for all component definitions
2. Check `apps/` for sample application structure
3. Deploy via `platform/argocd/bootstrap.yaml`
4. Practice manual TLS configuration
5. Create your own HTTPRoutes and Certificates
