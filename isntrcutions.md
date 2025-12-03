📌 TASK

Generate the following directory structure and the file contents inside.
All files must be valid YAML.
All kustomization.yaml files must use apiVersion: kustomize.config.k8s.io/v1beta1.

📁 REPO STRUCTURE TO GENERATE
k8s-gitops/
├── clusters/
│   ├── home-lab/
│   │   ├── apps.yaml
│   │   └── kustomization.yaml
│   ├── dev/
│   │   └── kustomization.yaml
│   ├── stage/
│   │   └── kustomization.yaml
│   └── prod/
│       └── kustomization.yaml
└── platform/
    ├── kustomization.yaml
    ├── argocd/
    │   ├── kustomization.yaml
    │   └── base/
    │       ├── kustomization.yaml
    │       └── namespace.yaml
    ├── metallb/
    │   ├── kustomization.yaml
    │   └── base/
    │       ├── kustomization.yaml
    │       ├── namespace.yaml
    │       └── metallb-config.yaml

📄 FILE CONTENTS TO GENERATE
1️⃣ clusters/home-lab/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - apps.yaml

2️⃣ clusters/home-lab/apps.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: platform
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/REPLACE_ME/k8s-gitops.git
    targetRevision: main
    path: platform
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true


(Replace repo URL)

3️⃣ Empty cluster placeholders

For dev, stage, prod:

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources: []

4️⃣ platform/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ./argocd
  - ./metallb

5️⃣ platform/argocd/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ./base

6️⃣ platform/argocd/base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: argocd
resources:
  - namespace.yaml

7️⃣ platform/argocd/base/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: argocd

8️⃣ platform/metallb/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ./base

9️⃣ platform/metallb/base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: metallb-system
resources:
  - namespace.yaml
  - metallb-config.yaml

🔟 platform/metallb/base/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: metallb-system

1️⃣1️⃣ platform/metallb/base/metallb-config.yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default
  namespace: metallb-system
spec:
  addresses:
    - 10.10.10.200-10.10.10.250
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
    - default

📌 ADDITIONAL REQUIREMENT

Also generate this bootstrap file (output separately):

platform/argocd/bootstrap.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: home-lab-root
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/REPLACE_ME/k8s-gitops.git
    targetRevision: main
    path: clusters/home-lab
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true

📌 OUTPUT FORMAT

Copilot should output all files in a single markdown block, with proper directory headings, like:

# clusters/home-lab/kustomization.yaml
<file content>

# clusters/home-lab/apps.yaml
<file content>
...

📌 DONE

Generate everything exactly as described. Only output the files and their contents.