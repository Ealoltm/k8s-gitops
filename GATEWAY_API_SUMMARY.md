# Gateway API GitOps - Files Summary

## Generated Files

### Platform Layer - Gateway API CRDs

**`platform/gateway-api/kustomization.yaml`**
- Root kustomization for gateway-api
- References `./base`

**`platform/gateway-api/base/kustomization.yaml`**
- Base kustomization
- References `gateway-api-application.yaml`

**`platform/gateway-api/base/gateway-api-application.yaml`**
- ArgoCD Application that installs official Gateway API v1.0.0 CRDs
- Source: `https://github.com/kubernetes-sigs/gateway-api.git`
- Installs to: `kube-system` namespace
- Features: Automated sync with prune & self-heal

### Platform Layer - Cilium Gateway Configuration

**`platform/ingress/cilium-gateway/kustomization.yaml`**
- Root kustomization for cilium gateway
- References `./base`

**`platform/ingress/cilium-gateway/base/kustomization.yaml`**
- Base kustomization with `cilium-gateway` namespace
- References `gatewayclass.yaml` and `gateway.yaml`

**`platform/ingress/cilium-gateway/base/gatewayclass.yaml`**
- Cilium GatewayClass registration
- Controller: `io.cilium/gateway-controller`
- Name: `cilium`

**`platform/ingress/cilium-gateway/base/gateway.yaml`**
- Cilium Gateway instance
- Listeners:
  - HTTP:80 (protocol: HTTP)
  - HTTPS:443 (protocol: HTTPS, TLS termination)
- Namespace: `cilium-gateway`

### Application Layer - Whoami Example

**`apps/kustomization.yaml`**
- Root kustomization for all applications
- Namespace: `default`
- References `whoami/kustomization.yaml`

**`apps/whoami/kustomization.yaml`**
- Whoami application kustomization
- Includes: deployment, service, httproute

**`apps/whoami/deployment.yaml`**
- 2 replicas of `containous/whoami:latest`
- Port: 80
- Resource limits: 50m CPU, 32Mi memory (request)

**`apps/whoami/service.yaml`**
- ClusterIP service
- Port: 80 → targetPort: http
- Selector: `app: whoami`

**`apps/whoami/httproute.yaml`**
- Gateway API HTTPRoute
- Parent: `cilium-gateway` (in `cilium-gateway` namespace)
- Hostname: `whoami.local`
- Path prefix: `/`
- Backend: `whoami` service on port 80

### Platform Root Update

**`platform/kustomization.yaml`** (UPDATED)
```yaml
resources:
  - ./argocd
  - ./metallb
  - ./gateway-api              # ← NEW
  - ./ingress/cilium-gateway   # ← NEW
```

### Documentation

**`GATEWAY_API.md`**
- Complete architecture overview
- Component explanations
- Integration guide
- TLS/cert-manager setup
- Troubleshooting guide
- Application development guide

## Deployment Flow

```
1. ArgoCD watches platform/ directory
2. Detects platform/kustomization.yaml changes
3. Applies resources in order:
   a. argocd/      (ArgoCD control plane)
   b. metallb/     (LoadBalancer IP provisioning)
   c. gateway-api/ (Gateway API CRDs)
   d. ingress/cilium-gateway/ (GatewayClass + Gateway)
4. Cilium operator reconciles Gateway
5. Applications with HTTPRoutes deployed
```

## API Versions Used

```yaml
gateway.networking.k8s.io/v1         # Gateway API v1
GatewayClass                           # API version 1
Gateway                                # API version 1
HTTPRoute                              # API version 1
```

## Cilium Integration

```
Controller: io.cilium/gateway-controller
Features:
  - Native Gateway API support via Cilium Operator
  - eBPF-based high-performance routing
  - Automatic LoadBalancer IP assignment via MetalLB
  - HTTP/HTTPS listener support
  - Path-based routing via HTTPRoute
  - TLS termination with cert-manager
```

## How to Use

### 1. Deploy Gateway API CRDs
```bash
# Automatic via ArgoCD
# Or manual:
kubectl apply -k platform/gateway-api/
```

### 2. Deploy Cilium GatewayClass and Gateway
```bash
# Automatic via ArgoCD
# Or manual:
kubectl apply -k platform/ingress/cilium-gateway/
```

### 3. Deploy Applications
```bash
# Automatic via ArgoCD
# Or manual:
kubectl apply -k apps/whoami/
```

### 4. Verify Deployment
```bash
# Check CRDs installed
kubectl get crds | grep gateway

# Check GatewayClass
kubectl get gatewayclass

# Check Gateway
kubectl -n cilium-gateway get gateway

# Check HTTPRoutes
kubectl get httproutes -A

# Test application
curl -H "Host: whoami.local" http://GATEWAY_IP
```

## Testing Commands

```bash
# Get Gateway external IP
kubectl -n cilium-gateway get gateway cilium-gateway -o wide

# Watch Gateway status
kubectl -n cilium-gateway get gateway cilium-gateway -w

# Check Cilium logs for gateway processing
kubectl -n kube-system logs -l k8s-app=cilium-operator --tail=50 | grep -i gateway

# Test HTTP routing
curl -v -H "Host: whoami.local" http://GATEWAY_IP/

# Test HTTPS (if cert configured)
curl -v -H "Host: whoami.local" https://GATEWAY_IP/

# Check HTTPRoute details
kubectl describe httproute whoami
```

## Next Steps

1. ✅ Gateway API CRDs installed
2. ✅ Cilium GatewayClass registered
3. ✅ Cilium Gateway deployed with HTTP/HTTPS listeners
4. ✅ Example whoami app with HTTPRoute included

To add more applications:
- Create `apps/app-name/` directory
- Add deployment, service, kustomization, httproute files
- Update `apps/kustomization.yaml`
- Commit and push - ArgoCD will auto-deploy

## Certificate Management (Future)

When adding cert-manager:
1. Create Certificate resource for `cilium-gateway-tls`
2. Reference Let's Encrypt ClusterIssuer
3. cert-manager will use Gateway API HTTP-01 challenge
4. Certificate automatically issued and renewed

See GATEWAY_API.md for detailed TLS setup instructions.
