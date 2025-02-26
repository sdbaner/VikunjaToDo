# Vikunja ToDo App

Helm v3.x  install command
```
kubectl create ns vikunja
helm install --name vikunja --namespace vikunja .
  --set VIKUNJA_DATABASE_PASSWORD=$SECRET_PGSQL
  --set VIKUNJA_KEYCLOAK_PASS=$SECRET_KCLK
  --values values.yaml
```


