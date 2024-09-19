kube-prometheus-stack
===========

Passos anteriores à instalação do chart `kube-prometheus-stack`, que inclui Prometheus e Grafana.

- [CRDs](#crds)
- [Secrets](#secrets)

## CRDs

Instalar CRDs. Atenção à versão da CRD. Usar versão compatível com o campo `spec.source.targetRevision` do arquivo [kube-prometheus-stack-argo-application.yaml](/kube-prometheus-stack/kube-prometheus-stack-argo-application.yaml).

```bash
kubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.71.0/example/prometheus-operator-crd/monitoring.coreos.com_alertmanagerconfigs.yaml
kubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.71.0/example/prometheus-operator-crd/monitoring.coreos.com_alertmanagers.yaml
kubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.71.0/example/prometheus-operator-crd/monitoring.coreos.com_podmonitors.yaml
kubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.71.0/example/prometheus-operator-crd/monitoring.coreos.com_probes.yaml
kubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.71.0/example/prometheus-operator-crd/monitoring.coreos.com_prometheusagents.yaml
kubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.71.0/example/prometheus-operator-crd/monitoring.coreos.com_prometheuses.yaml
kubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.71.0/example/prometheus-operator-crd/monitoring.coreos.com_prometheusrules.yaml
kubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.71.0/example/prometheus-operator-crd/monitoring.coreos.com_scrapeconfigs.yaml
kubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.71.0/example/prometheus-operator-crd/monitoring.coreos.com_servicemonitors.yaml
kubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.71.0/example/prometheus-operator-crd/monitoring.coreos.com_thanosrulers.yaml
```

> :warning: Muito cuidado ao atualizar CRDs. Caso as CRDs sejam deletadas, serão deletados todos os recursos — cujo campo "kind" for a CRD — de todos os namespaces do cluster Kubernetes.

## Secrets

Criar namespace onde a ferramenta será instalada. Pular etapa caso namespace já exista.

```
kubectl create namespace monitoring
```

Criação do usuário admin do Grafana. Substitua as expressões "SUBSTITUA_POR_UM_NOME_DE_USUARIO" e "SUBSTITUA_POR_UMA_SENHA".

```
kubectl -n monitoring apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: grafana-admin
type: Opaque
stringData:
  admin-user: SUBSTITUA_POR_UM_NOME_DE_USUARIO
  admin-password: SUBSTITUA_POR_UMA_SENHA
EOF
```

Configuração do email para envio de convites a novos usuários por meio do Grafana. Substitua as expressões "SUBSTITUA_POR_UM_EMAIL" e "SUBSTITUA_PELA_SENHA_DO_EMAIL".

```
kubectl -n monitoring apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: grafana-smtp
type: Opaque
stringData:
  user: SUBSTITUA_POR_UM_EMAIL
  password: SUBSTITUA_PELA_SENHA_DO_EMAIL
EOF
```

Configuração do contact point do Telegram para o Alertmanager. Necessário já ter um bot criado no telegram.
Substitua os Campos "SUBSTITUA_POR_UM_CHAT_ID" e "SUBSTITUA_POR_UM_TOKEN".

```
kubectl -n monitoring apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: telegram-bot-secret
type: Opaque
stringData:
  bot_token: SUBSTITUA_POR_UM_TOKEN
EOF
```

```
kubectl -n monitoring apply -f - <<EOF
apiVersion: monitoring.coreos.com/v1alpha1
kind: AlertmanagerConfig
metadata:
  name: alertmanagerconfig
  namespace: monitoring
  labels:
    alertmanagerConfig: global
spec:
  route:
    receiver: 'telegram-receiver'
  receivers:
  - name: 'telegram-receiver'
    telegramConfigs:
    - apiURL: 'https://api.telegram.org'
      botToken:
        key: bot_token
        name: telegram-bot-secret
      chatID: SUBSTITUA_POR_UM_CHAT_ID
EOF
```
