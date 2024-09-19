# Create secrets

openssl rand -base64 42

exemplo de saida

``` WnsoGvMXoyRYyA/dDtfY7Qmxfjx/M5hETAkUPZyvC6TJpdhoeZa3vwUN ```

Criando a secrets no k8s

```  kubectl create secret generic superset-secret-key --from-literal=SUPERSET_SECRET_KEY='WnsoGvMXoyRYyA/dDtfY7Qmxfjx/M5hETAkUPZyvC6TJpdhoeZa3vwUN' \  -n <seu-namespace>
```