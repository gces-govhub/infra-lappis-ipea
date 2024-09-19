airflow
===========

Instalação do Airflow em um cluster Kubernetes a partir do ArgoCD. Antes, é necessário fazer o deploy do MinIO.

- [Secrets](#secrets)
- [Fernet Key](#fernet-key)
- [PostgreSQL](#postgresql)
- [Setup](#setup)

## Secrets

Criar namespace onde a ferramenta será instalada. Pular etapa caso namespace já exista.

```bash
kubectl create namespace airflow
```

Criação do usuário admin do Airflow. Substitua os campos: SUBSTITUA_POR_UM_USERNAME, SUBSTITUA_POR_UM_EMAIL, SUBSTITUA_POR_UM_FIRSTNAME, SUBSTITUA_POR_UM_LASTNAME e SUBSTITUA_POR_UM_PASSWORD.

```yaml
kubectl -n airflow apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: airflow-webserver-admin-credentials
type: Opaque
stringData:
  WEBSERVER_DEFAULT_USER_ROLE: Admin
  WEBSERVER_DEFAULT_USER_USERNAME: SUBSTITUA_POR_UM_USERNAME
  WEBSERVER_DEFAULT_USER_EMAIL: SUBSTITUA_POR_UM_EMAIL
  WEBSERVER_DEFAULT_USER_FIRSTNAME: SUBSTITUA_POR_UM_FIRSTNAME
  WEBSERVER_DEFAULT_USER_LASTNAME: SUBSTITUA_POR_UM_LASTNAME
  WEBSERVER_DEFAULT_USER_PASSWORD: SUBSTITUA_POR_UM_PASSWORD
EOF
```

Criação de secret para envio de emails. Substitua os campos por valores apropriados. Obs: a senha "dfpceiqvijnrbvxt" (app password do gmail) é só um exemplo, não é utilizada para nada.

```yaml
kubectl -n airflow apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: airflow-smtp
type: Opaque
stringData:
  host: "smtp.gmail.com"
  starttls: "True"
  ssl: "False"
  user: "infraestrutura.lappis@gmail.com"
  password: "dfpceiqvijnrbvxt"
  port: "587"
  mail_from: "infraestrutura.lappis@gmail.com"
EOF
```

Criação de Secret para acesso aos repositórios das dags e dos plugins. Substitua "SUBSTITUIR_POR_UM_PERSONAL_ACCESS_TOKEN".

```yaml
kubectl -n airflow apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: airflow-git-credentials
type: Opaque
stringData:
  GIT_SYNC_USERNAME: nonexistant
  GIT_SYNC_PASSWORD: SUBSTITUIR_POR_UM_PERSONAL_ACCESS_TOKEN
  GITSYNC_USERNAME: nonexistant
  GITSYNC_PASSWORD: SUBSTITUIR_POR_UM_PERSONAL_ACCESS_TOKEN
EOF
```

Criação da secret key do webserver do Airflow. Substitua "SUBSTITUA_POR_UMA_SECRET_KEY".

```yaml
kubectl -n airflow apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: airflow-webserver-secret-key
type: Opaque
stringData:
  webserver-secret-key: SUBSTITUA_POR_UMA_SECRET_KEY
EOF
```

Caso o componente Redis do Helm chart do Airflow esteja ativado, caso o executor utilizado seja o CeleryExecutor ou o CeleryKubernetesExecutor, os docs do Airflow recomendam configurar uma senha estática para o Redis. Para configurar a senha estática, é necessário criar um Secret para a senha e um Secret para o broker url (ambos os Secrets devem conter a senha). Substitua "SUBSTITUA_POR_UMA_SENHA_PARA_O_REDIS".

```yaml
kubectl -n airflow apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: airflow-redis-password
type: Opaque
stringData:
  password: SUBSTITUA_POR_UMA_SENHA_PARA_O_REDIS
---
apiVersion: v1
kind: Secret
metadata:
  name: airflow-broker-url
type: Opaque
stringData:
  connection: redis://:SUBSTITUA_POR_UMA_SENHA_PARA_O_REDIS@airflow-redis:6379/0
EOF
```

## Fernet Key

A chave Fernet é utilizada para encriptar as conexões no banco de dados, impedindo que sejam alteradas sem o uso da chave Fernet. Logo, para rotacionar a Fernet key, é necessário reencriptar partes do banco de dados do Airflow. A documentação do Airflow ensina como atualizar a Fernet key no seguinte link: <https://airflow.apache.org/docs/apache-airflow/2.7.3/security/secrets/fernet.html#rotating-encryption-keys>.

### Gerar chave Fernet
Para gerar uma chave Fernet, basta rodar o seguinte código em Python.

```python
from cryptography.fernet import Fernet

fernet_key = Fernet.generate_key()
print(fernet_key.decode())  # your fernet_key, keep it in secured place!
```

### Kubernetes Secret
Após gerar a chave fernet, substituir o campo "SUBSTITUIR_POR_FERNET_KEY" e criar o recurso Secret.

```yaml
kubectl -n airflow apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: airflow-fernet-key
type: Opaque
stringData:
  fernet-key: SUBSTITUIR_POR_FERNET_KEY
EOF
```

### Rotacionar chave Fernet

Ver <https://airflow.apache.org/docs/apache-airflow/2.7.3/security/secrets/fernet.html#rotating-encryption-keys>

## PostgreSQL

É necessário criar uma database e um usuário que o Airflow vai usar para acessar a database. Substitua "NOME_DA_DATABASE", "NOME_DE_USUARIO", "SENHA".


```
CREATE DATABASE NOME_DA_DATABASE;

CREATE USER NOME_DE_USUARIO WITH PASSWORD 'SENHA';

GRANT ALL PRIVILEGES ON DATABASE NOME_DA_DATABASE TO NOME_DE_USUARIO;

-- PostgreSQL 15 requires additional privileges:
USE NOME_DA_DATABASE;

GRANT ALL ON SCHEMA public TO NOME_DE_USUARIO;
```

Criação do secret para acesso ao banco de dados PostgreSQL. Substitua as expressões "NOME_DE_USUARIO", "SENHA", "HOST", "PORT" e "NOME_DA_DATABASE".

```yaml
kubectl -n airflow apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: airflow-metadata
type: Opaque
stringData:
  connection: postgresql://NOME_DE_USUARIO:SENHA@HOST:PORT/NOME_DA_DATABASE
EOF
```

Mais info: <https://airflow.apache.org/docs/apache-airflow/stable/howto/set-up-database.html#setting-up-a-postgresql-database>

## Setup
Há três formas de instalação, documentadas a seguir.

### Via Helm

Clonar este repositório.

```bash
git clone git@gitlab.com:lappis-unb/decidimbr/infra/continuous-deployment.git
```

Navegar até a pasta 'continuous-deployment/airflow'.

```bash
cd continuous-deployment
cd airflow
```

Navegar até a pasta 'homolog' ou 'production', a depender do ambiente.

```bash
cd production
# ou
cd homolog
```

Clonar o repositório do nosso chart do Airflow para a pasta './charts/airflow'.

```bash
mkdir -p charts/airflow

git clone git@gitlab.com:lappis-unb/decidimbr/infra/charts/airflow.git ./charts/airflow
```

Dar permissão para execução do script './helm_post_renderer.sh'.

```bash
chmod +x ./helm_post_renderer.sh
```

Instalar com Helm.

```bash
helm upgrade --install --wait \
  airflow ./charts/airflow --namespace airflow \
  --post-renderer ./helm_post_renderer.sh
```

### Via ArgoCD para o ambiente de homolog.

```bash
kubectl -n argocd apply -f - <<EOF
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: airflow-install
  finalizers:
  - resources-finalizer.argocd.argoproj.io
spec:
  destination:
    name: ''
    namespace: airflow
    server: 'https://kubernetes.default.svc'
  source:
    path: 'airflow/homolog'
    repoURL: 'https://gitlab.com/lappis-unb/decidimbr/infra/continuous-deployment.git'
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

### Via ArgoCD para o ambiente de produção.

```bash
kubectl -n argocd apply -f - <<EOF
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: airflow-install
  finalizers:
  - resources-finalizer.argocd.argoproj.io
spec:
  destination:
    name: ''
    namespace: airflow
    server: 'https://kubernetes.default.svc'
  source:
    path: 'airflow/production'
    repoURL: 'https://gitlab.com/lappis-unb/decidimbr/infra/continuous-deployment.git'
    targetRevision: v0.3.0
    plugin:
      name: kustomized-helm
  sources: []
  project: default
  syncPolicy:
    syncOptions:
    - CreateNamespace=true
EOF
```
