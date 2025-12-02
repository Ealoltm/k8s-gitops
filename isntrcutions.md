✅ VS Code Copilot Prompt — GitOps Repository (Multi-Env, Platform Phase)

You are generating a Kubernetes GitOps repository structure for Argo CD using a multi-environment “app-of-apps” pattern.
Follow these exact requirements:

🎯 Overall Goal

Create a complete folder + file structure for a GitOps repository called k8s-gitops, using:

Argo CD as the GitOps controller

Multi-environment layout (home-lab, dev, stage, prod)

Platform components defined in Git

Argo CD bootstrapped via a root Application

MetalLB config managed via GitOps

Argo CD managing its own Helm chart

Declarative, production-grade layout

📁 Required Directory Structure

Create the following folder tree:

k8s-gitops/
  clusters/
    home-lab/
      apps.yaml
    dev/
      apps.yaml
    stage/
      apps.yaml
    prod/
      apps.yaml

  platform/
    metallb/
      base/
        metallb-config.yaml
      kustomization.yaml

    argocd/
      base/
        argocd-application.yaml
      kustomization.yaml


(Do not generate empty folders — each folder must contain the proper files.)

📦 MetalLB GitOps Manifests

Inside platform/metallb/base/metallb-config.yaml, generate the following exact manifests:

IPAddressPool named kube-pool

L2Advertisement named kube-l2adv

Namespace: metallb-system

IP range: 10.10.10.200-10.10.10.250

🚀 Argo CD GitOps (Self-Managed)

Inside platform/argocd/base/argocd-application.yaml, generate an Argo CD Application that:

Installs the argo-cd Helm chart

From repo https://argoproj.github.io/argo-helm

Into namespace argocd

Uses LoadBalancer type with IP 10.10.10.200

Sets configs.params.server.insecure=true

Enables automated sync (prune + self-heal)

🧩 Root App-of-Apps (Environment Scoped)

Inside clusters/home-lab/apps.yaml, generate:

One Application referencing platform/metallb/base

One Application referencing platform/argocd/base

Both pointing to the Git repo (placeholder variable: <GITHUB_URL>)

Include:

syncPolicy.automated.prune = true

syncPolicy.automated.selfHeal = true

syncOptions = ["CreateNamespace=true"]

📝 Additional Requirements

Use valid YAML, no placeholders except <GITHUB_URL>

All manifests must pass Kubernetes + ArgoCD validation

Avoid indentation mistakes

Follow Argo CD best practices for multi-environment GitOps

Output EVERYTHING in a single response:

Directory tree

All YAML files

All kustomization.yaml files

Output Format

Respond with:

Repository folder tree (Markdown code block)

Each file as a separate YAML code block with its path above it

Do NOT explain. Only generate the structure + files.