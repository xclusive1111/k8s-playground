# Prometheus Operator

In this lab, we will install Prometheus Operator that helps with monitoring cluster components and resources.

![](/home/sondv/.config/marktext/images/2022-08-15-11-05-38-image.png)

### Deploy kube-prometheus

Unzip the `manifest/zip` file under the `deployments` directory

```shell
# Create the namespace and CRDs, and then wait for them to be availble before creating the remaining resources
kubectl create -f manifests/setup

# Wait until the "servicemonitors" CRD is created. The message "No resources found" means success in this context.
until kubectl get servicemonitors --all-namespaces ; do date; sleep 1; echo ""; done

kubectl create -f manifests/
```

Check deployment status and ensure all pods are up and running

```shell
NAME                                       READY   STATUS    RESTARTS   AGE
pod/alertmanager-main-0                    2/2     Running   0          4h21m
pod/blackbox-exporter-69684688c9-grnm8     3/3     Running   0          7h3m
pod/grafana-7c89b88867-zb7ls               1/1     Running   0          19m
pod/kube-state-metrics-98bdf47b9-tqq6z     3/3     Running   0          7h3m
pod/node-exporter-4nm9k                    2/2     Running   0          4h9m
pod/prometheus-adapter-57b54f47f7-b5gzl    1/1     Running   0          4h15m
pod/prometheus-adapter-57b54f47f7-fz9d5    1/1     Running   0          4h15m
pod/prometheus-k8s-0                       2/2     Running   0          4h5m
pod/prometheus-operator-5c89f7b7df-kl7g4   2/2     Running   0          4h29m

NAME                            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
service/alertmanager-main       ClusterIP   10.32.0.198   <none>        9093/TCP,8080/TCP            7h3m
service/alertmanager-operated   ClusterIP   None          <none>        9093/TCP,9094/TCP,9094/UDP   4h21m
service/blackbox-exporter       ClusterIP   10.32.0.35    <none>        9115/TCP,19115/TCP           7h3m
service/grafana                 ClusterIP   10.32.0.166   <none>        3000/TCP                     7h3m
service/kube-state-metrics      ClusterIP   None          <none>        8443/TCP,9443/TCP            7h3m
service/node-exporter           ClusterIP   None          <none>        9100/TCP                     7h3m
service/prometheus-adapter      ClusterIP   10.32.0.153   <none>        443/TCP                      7h3m
service/prometheus-k8s          ClusterIP   10.32.0.161   <none>        9090/TCP,8080/TCP            7h3m
service/prometheus-operated     ClusterIP   None          <none>        9090/TCP                     4h5m
service/prometheus-operator     ClusterIP   None          <none>        8443/TCP                     7h3m

NAME                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/node-exporter   1         1         1       1            1           kubernetes.io/os=linux   4h9m

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/blackbox-exporter     1/1     1            1           7h3m
deployment.apps/grafana               1/1     1            1           19m
deployment.apps/kube-state-metrics    1/1     1            1           7h3m
deployment.apps/prometheus-adapter    2/2     2            2           4h15m
deployment.apps/prometheus-operator   1/1     1            1           4h29m

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/blackbox-exporter-69684688c9     1         1         1       7h3m
replicaset.apps/grafana-7c89b88867               1         1         1       19m
replicaset.apps/kube-state-metrics-98bdf47b9     1         1         1       7h3m
replicaset.apps/prometheus-adapter-57b54f47f7    2         2         2       4h15m
replicaset.apps/prometheus-operator-5c89f7b7df   1         1         1       4h29m

NAME                                 READY   AGE
statefulset.apps/alertmanager-main   1/1     4h21m
statefulset.apps/prometheus-k8s      1/1     4h5m
```

Our Grafana dashboad will be running behind Nginx ingress controller. The default **grafana** root url has been changed to `/grafana`. If you wish to use a different sub path, make changes accordingly from the file `grafana-config.yaml`

Create a grafana ingress:

```shell
kubectl apply -f deployments/grafana.yaml
```

### Grafana Dashboard

Access the Grafana dashboard at [http://lb-0/grafana]() and add some metrics:

![](/home/sondv/.config/marktext/images/2022-08-15-10-53-26-image.png)

Reference link: [Quick Start - Prometheus Operator (prometheus-operator.dev)](https://prometheus-operator.dev/docs/prologue/quick-start/)
