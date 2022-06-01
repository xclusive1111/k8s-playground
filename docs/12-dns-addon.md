# Deploying the DNS Cluster Add-on

In this lab you will deploy the [DNS add-on](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) which provides DNS based service discovery, backed by [CoreDNS](https://coredns.io/), to applications running inside the Kubernetes cluster.

## The DNS Cluster Add-on

Deploy the `coredns` cluster add-on:`blockaffinities`

```
kubectl apply -f ./deployments/coredns-1.9.2.yaml
```

```
serviceaccount/coredns created
clusterrole.rbac.authorization.k8s.io/system:coredns created
clusterrolebinding.rbac.authorization.k8s.io/system:coredns created
configmap/coredns created
deployment.apps/coredns created
service/kube-dns created
```

List the pods created by the `kube-dns` deployment:

```
kubectl get pods -l k8s-app=kube-dns -n kube-system
```

```
NAME                       READY   STATUS    RESTARTS   AGE
coredns-6cd56d4df4-6h4rw   1/1     Running   0          52s
coredns-6cd56d4df4-lm4j4   1/1     Running   0          52s
```

## Verification

### Lookup DNS

Create a `busybox` deployment:

```
kubectl run busybox --image=busybox --command -- sleep 3600
```

List the pod created by the `busybox` deployment:

```
kubectl get pods -l run=busybox
```

```
NAME      READY   STATUS    RESTARTS   AGE
busybox   1/1     Running   0          3s
```

Retrieve the full name of the `busybox` pod:

```
POD_NAME=$(kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}")
```

Execute a DNS lookup for the `kubernetes` service inside the `busybox` pod:

```
kubectl exec -ti $POD_NAME -- nslookup kubernetes
```

```
Server:    10.32.0.10
Address 1: 10.32.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes
Address 1: 10.32.0.1 kubernetes.default.svc.cluster.local
```

### Test the service by accessing it from another Pod

Create an nginx deployment:

```shell
kubectl create deployment nginx --image=nginx
```

```
deployment.apps/nginx created
```

Expose the Deployment through a Service called `nginx`:

```shell
kubectl expose deployment nginx --port=80
```

```
service/nginx exposed
```

The above commands create a Deployment with an nginx Pod and expose the Deployment through a Service named `nginx`. The `nginx` Pod and Deployment are found in the `default` namespace.

```shell
kubectl get svc,pod
```

```
NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.32.0.1     <none>        443/TCP   3d
service/nginx        ClusterIP   10.32.0.122   <none>        80/TCP    61m

NAME                        READY   STATUS    RESTARTS   AGE
pod/nginx-8f458dc5b-r2qr7   1/1     Running   0          3m32s
```

You should be able to access the new `nginx` service from other Pods. To access the `nginx` Service from another Pod in the `default` namespace, start a busybox container:

```shell
kubectl run box1 -it --rm --image busybox -- sh
```

Inside the shell, run the following command:

```
wget --spider --timeout=1 nginx
```

```
Connecting to nginx (10.32.0.122:80)
remote file exists
```

Next: [Smoke Test](13-smoke-test.md)
