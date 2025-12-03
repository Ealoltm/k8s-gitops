# Gateway API GitOps Structure

This directory contains the complete Gateway API configuration for Cilium-based ingress management in a GitOps-driven Kubernetes cluster.

## Architecture Overview

```
┌─────────────────────────────────────────┐
│         ArgoCD (GitOps)                  │
└────────────┬────────────────────────────┘
             │
             ├─── platform/gateway-api ──────┐
             │                               │
             │  Installs official Gateway    │
             │  API CRDs from upstream       │
             │                               │
             └───────────┬───────────────────┘
                         │
         ┌───────────────┴───────────────────┐
         │                                   │
    ┌────▼─────────────┐          ┌─────────▼──────┐
    │ GatewayClass     │          │    Gateway     │
    │ (cilium)         │          │ (cilium-gw)    │
    └────┬─────────────┘          └─────────┬──────┘
         │                                  │
         └──────────────┬───────────────────┘
                        │
         ┌──────────────┴──────────────┐
         │                             │
    ┌────▼──────────┐        ┌────────▼────┐
    │   HTTP:80     │        │  HTTPS:443  │
    │   (Cilium LB) │        │  (cert-mgr) │
    └────┬──────────┘        └────────┬────┘
         │                            │
         └────────────┬───────────────┘
                      │
         ┌────────────┴─────────────┐
         │                          │
    ┌────▼──────────┐    ┌─────────▼────┐
    │   HTTPRoute   │    │   HTTPRoute   │
    │   (whoami)    │    │  (user apps)  │
    └────┬──────────┘    └──────────────┘
         │
    ┌────▼──────────┐
    │   Service     │
    │   (whoami)    │
    └───────────────┘
```

## Directory Structure

```
platform/
├── gateway-api/                           # Official Gateway API CRDs
│   ├── kustomization.yaml
│   └── base/
│       ├── kustomization.yaml
│       └── gateway-api-application.yaml   # ArgoCD Application
│
└── ingress/
    └── cilium-gateway/                    # Cilium Gateway API config
        ├── kustomization.yaml
        └── base/
            ├── kustomization.yaml
            ├── gatewayclass.yaml          # Cilium controller registration
            └── gateway.yaml               # HTTP/HTTPS listeners

apps/
├── kustomization.yaml
└── whoami/                                # Example app using HTTPRoute
    ├── kustomization.yaml
    ├── deployment.yaml
    ├── service.yaml
    └── httproute.yaml
```

## Components

### 1. Gateway API CRDs (platform/gateway-api/)

Installs official Gateway API v1.0.0 CRDs from upstream Kubernetes SIGs.

**ArgoCD Application:**
```yaml
spec:
  source:
    repoURL: https://github.com/kubernetes-sigs/gateway-api.git
    targetRevision: v1.0.0
    path: config/crd
```

**Resources installed:**
- `gateway.networking.k8s.io/v1`
- `gatewayclass.networking.k8s.io/v1`
- `httproute.gateway.networking.k8s.io/v1`
- `tcproute.gateway.networking.k8s.io/v1`
- And other Gateway API resources

### 2. GatewayClass (platform/ingress/cilium-gateway/base/gatewayclass.yaml)

Registers Cilium as the controller for Gateway API resources.

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: cilium
spec:
  controllerName: io.cilium/gateway-controller
```

**What this means:**
- Any Gateway referencing `gatewayClassName: cilium` will be managed by Cilium
- Cilium automatically provisions load balancing for HTTP/HTTPS traffic
- Cilium uses eBPF for high-performance traffic routing

### 3. Gateway (platform/ingress/cilium-gateway/base/gateway.yaml)

Defines the load balancer instance with HTTP and HTTPS listeners.

```yaml
spec:
  gatewayClassName: cilium
  listeners:
    - name: http
      port: 80
      protocol: HTTP
    - name: https
      port: 443
      protocol: HTTPS
      tls:
        mode: Terminate
        certificateRefs:
          - name: cilium-gateway-tls
```

**Listeners:**
- **HTTP:80** - Unsecured traffic (redirected to HTTPS by default)
- **HTTPS:443** - Secured traffic with TLS certificates from cert-manager

### 4. HTTPRoute (apps/whoami/httproute.yaml)

Application ingress definition using Gateway API.

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: whoami
spec:
  parentRefs:
    - name: cilium-gateway
      namespace: cilium-gateway
  hostnames:
    - whoami.local
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: whoami
          port: 80
```

**How it works:**
1. HTTPRoute references the Gateway (parent) in `cilium-gateway` namespace
2. Specifies hostnames and path matching rules
3. Routes traffic to backend Service (whoami:80)
4. Cilium automatically loads this configuration

## Integration with Platform

The Gateway API is integrated into the platform deployment chain:

**platform/kustomization.yaml:**
```yaml
resources:
  - ./argocd
  - ./metallb
  - ./gateway-api          # ← Installs CRDs first
  - ./ingress/cilium-gateway
```

**Deployment order (ArgoCD respects this):**
1. ArgoCD installs
2. MetalLB installs (provides LoadBalancer service type)
3. Gateway API CRDs installed
4. Cilium GatewayClass + Gateway deployed
5. Apps (whoami, etc.) with HTTPRoutes deployed

## TLS/HTTPS Setup (cert-manager)

To enable HTTPS, you need to:

### 1. Create a Certificate for the Gateway
```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: cilium-gateway-tls
  namespace: cilium-gateway
spec:
  secretName: cilium-gateway-tls
  commonName: "*.example.com"
  dnsNames:
    - example.com
    - "*.example.com"
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
```

### 2. Create an Issuer
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
      - http01:
          gatewayHTTPRoute:
            parentRefs:
              - name: cilium-gateway
                namespace: cilium-gateway
```

The `http01` solver uses the Gateway API to validate HTTPS certificates via HTTP-01 challenge.

## Adding New Applications

To add a new application using the Gateway:

### 1. Create application files

```bash
mkdir -p apps/my-app
cat > apps/my-app/kustomization.yaml
cat > apps/my-app/deployment.yaml
cat > apps/my-app/service.yaml
cat > apps/my-app/httproute.yaml
```

### 2. Define HTTPRoute

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: my-app
spec:
  parentRefs:
    - name: cilium-gateway
      namespace: cilium-gateway
  hostnames:
    - my-app.example.com
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: my-app
          port: 8080
```

### 3. Add to apps/kustomization.yaml

```yaml
resources:
  - whoami/kustomization.yaml
  - my-app/kustomization.yaml
```

### 4. Commit and push
```bash
git add .
git commit -m "Add my-app with Gateway API HTTPRoute"
git push
```

ArgoCD will automatically sync and deploy your app.

## Verification

### Check Gateway API CRDs
```bash
kubectl get crds | grep gateway
# Should show:
# - gatewayclasses.gateway.networking.k8s.io
# - gateways.gateway.networking.k8s.io
# - httproutes.gateway.networking.k8s.io
# - tcproutes.gateway.networking.k8s.io
```

### Check Cilium GatewayClass
```bash
kubectl get gatewayclass
# Output:
# NAME     CONTROLLER
# cilium   io.cilium/gateway-controller
```

### Check Gateway
```bash
kubectl -n cilium-gateway get gateways
# Output:
# NAME              CLASS    ADDRESS      PORTS   AGE
# cilium-gateway    cilium   10.10.10.x   80,443  5m
```

### Check HTTPRoutes
```bash
kubectl get httproutes -A
# Should list all HTTPRoute resources from apps/
```

### Test application routing
```bash
# Get Gateway external IP
GATEWAY_IP=$(kubectl -n cilium-gateway get gateway cilium-gateway -o jsonpath='{.status.addresses[0].value}')

# Test whoami app
curl -H "Host: whoami.local" http://$GATEWAY_IP

# Should return:
# Hostname: whoami-xxxxx
# IP: 10.x.x.x
# ...
```

## Troubleshooting

### Gateway doesn't get LoadBalancer IP
```bash
kubectl -n cilium-gateway describe gateway cilium-gateway
# Check for events/conditions
# Ensure MetalLB is running: kubectl -n metallb-system get pods
```

### HTTPRoute not routing traffic
```bash
# Check HTTPRoute status
kubectl describe httproute whoami

# Check Cilium Gateway status
kubectl -n kube-system logs -l k8s-app=cilium-operator | grep gateway

# Verify backend service exists
kubectl get svc whoami
```

### TLS certificate not issued
```bash
# Check certificate status
kubectl -n cilium-gateway describe cert cilium-gateway-tls

# Check cert-manager logs
kubectl -n cert-manager logs -l app=cert-manager

# Check challenge
kubectl get challenge -A
```

## References

- [Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/)
- [Cilium Gateway API Support](https://docs.cilium.io/en/stable/network/servicemesh/gateway-api/)
- [cert-manager HTTP-01 Challenge](https://cert-manager.io/docs/challenges/http01/)
