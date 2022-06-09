# Kubernetes Dashboard

In this lab, we will install the kubernetes dashboard that provides web-based UI for Kubernetes cluster. It allows users to manage applications running in the cluster and troubleshoot them, as well as manage the cluster itself.

![](/home/sondv/.config/marktext/images/2022-06-09-11-41-31-image.png)

The kubernetes dashboards consists of 3 components: `metrics-server`, `metrics-scraper` and `dashboard`

```shell
kubectl apply -f \
 ./deployments/metrics-server.yaml \
 ./deployments/kubernetes-dashboard.yaml \
 ./deployments/dashboard-admin.yaml
```

To access Dashboard, you must create a access token using the Service Account `admin-user` created above:

```shell
kubectl -n kube-system create token admin-user
```

```
eyJhbGciOiJSUzI1NiIsImtpZCI6InYyYnJuTWx3R01ZLWNHaklKdVN4VlNTN0ozMWdMTks2cG5wQXl3YW1tSkEifQ.eyJhdWQiOlsiaHR0cHM6Ly8xOTIuMTY4LjMzLjMwOjY0NDMiXSwiZXhwIjoxNjU0NzUzNTI4LCJpYXQiOjE2NTQ3NDk5MjgsImlzcyI6Imh0dHBzOi8vMTkyLjE2OC4zMy4zMDo2NDQzIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJhZG1pbi11c2VyIiwidWlkIjoiNjRiMzEyYTktYjFlNC00Y2I1LWJlMmEtNGU0YzA4YjViNjk4In19LCJuYmYiOjE2NTQ3NDk5MjgsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTphZG1pbi11c2VyIn0.eNqAWZstTNtinZQklEnXbRRMG5tLGXZqL5s-uawJmTrqiO8zirek-PUI_ZU1gZNWdbaSSh8Q0RaGLOWQ1cU-7ZzteOoQdaO0hw5khXO6sbDQH_u7rWUvY_DC1uWzV4WKueMjgrMh9BoJ4frKyRjukv_2mHsCtlaAflGBLJ1blEdmvHpImaEX1aLSNY4cYMVPk3kemjJx-BLIlZfTY-P767NIVcDIpseTNLLtvOqXC8jgdDOzzDAE5qkla5pMXsmROzpDgPIPiqFHTBH6bW6C_XvqZkCbwjybmtudT1tbjBUnnfL5sqLt3W8m5tOaOjHJX_Y409-CeWhu99y2QNEPTg
```

Now use this token to access Dashboard at:

[https://lb-0/#/workloads?namespace=default]()
