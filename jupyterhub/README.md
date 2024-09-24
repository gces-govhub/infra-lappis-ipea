jupyterhub
===========

## Setup
Há três formas de instalação, documentadas a seguir.

### Via Helm

Clonar este repositório.

```bash
git clone git@gitlab.com:lappis-unb/gest-odadosipea/infra-lappis-ipea.git
```

Navegar até a pasta 'infra-lappis-ipea/jupyterhub'.

```bash
cd infra-lappis-ipea
cd jupyterhub
```

Navegar até a pasta 'homolog' ou 'production', a depender do ambiente.

```bash
cd production
# ou
cd homolog
```

Inflar o Helm chart do jupyterhub.

```bash
kubectl kustomize . --enable-helm --load-restrictor LoadRestrictionsNone
```

Dar permissão para execução do script './helm_post_renderer.sh'.

```bash
chmod +x ./helm_post_renderer.sh
```

Instalar com Helm.

```bash
helm upgrade --install --wait \
  jupyterhub ./charts/jupyterhub --namespace jupyterhub \
  --post-renderer ./helm_post_renderer.sh
```

### Via ArgoCD para o ambiente de homolog.

```bash
kubectl -n argocd apply -f - <<EOF
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: jupyterhub-install
  finalizers:
  - resources-finalizer.argocd.argoproj.io
spec:
  destination:
    name: ''
    namespace: jupyterhub
    server: 'https://kubernetes.default.svc'
  source:
    path: 'jupyterhub/homolog'
    repoURL: 'https://gitlab.com/lappis-unb/gest-odadosipea/infra-lappis-ipea.git'
    targetRevision: main
    plugin:
      name: kustomized-helm
  sources: []
  project: default
  syncPolicy:
    syncOptions:
    - CreateNamespace=true
EOF
```
