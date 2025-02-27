# Observability

Step 1: Install Prometheus and Alertmanager using Helm
Use the kube-prometheus-stack Helm chart which includes Prometheus, Alertmanager, and Grafana, along with several pre-configured dashboards and alerting rules

```
helm upgrade --install kube-prometheus-stack prometheus-community/kube-prometheus-stack -f values.yaml
```

![Screen Shot 2025-02-26 at 11 29 29 PM](https://github.com/user-attachments/assets/b28e6bf1-010f-4f1b-b341-2858afebab48)


## Alerts

- check if vikunja url is down
- api url is down
- keycloak service is down
- postgres is down
- keycloak multiple auth failures

