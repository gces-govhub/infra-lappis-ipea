kube-vip
===========

Configuração de IPs virtuais para o control plane do cluster.

## Setup

Há três formas de instalação, documentadas a seguir.

### Via kubectl usando kustomize para o ambiente de homolog

Para homolog:

```bash
kubectl apply -k kube-vip/homolog
```

### Via ArgoCD para o ambiente de homolog.

Substituir as expressões começadas com "SUBSTITUIR_".

```bash
kubectl -n argocd apply -f - <<EOF
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: kube-vip-install
  finalizers:
  - resources-finalizer.argocd.argoproj.io
spec:
  destination:
    name: ''
    namespace: kube-system
    server: 'https://kubernetes.default.svc'
  source:
    path: 'kube-vip/homolog'
    repoURL: 'SUBSTITUIR_PELA_URL_DO_REPO'
    targetRevision: main
  sources: []
  project: default
  syncPolicy:
    syncOptions:
    - CreateNamespace=true
EOF
```
