# Provisioning Pod Network Routes

Pods scheduled to a node receive an IP address from the node's Pod CIDR range. At this point pods can not communicate with other pods running on different nodes due to missing network [routes](https://cloud.google.com/compute/docs/vpc/routes).

In this lab you will create a route for each worker node that maps the node's Pod CIDR range to the node's internal IP address.

> There are [other ways](https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-achieve-this) to implement the Kubernetes networking model.

Run a sample pods

```shell
kubectl run box1 -it --rm --image busybox -- sh
kubectl run box2 -it --rm --image busybox -- sh
```

Ensure the both pods are running

```shell
kubectl get pods -o wide
```

 At this point, two boxes cannot ping each other.

## Deploy Weave Network

Run the following only once on the `master` node:

```shell
CLUSTER_CIDR=10.200.0.0/16
kubectl --kubeconfig admin.kubeconfig apply -f "https://cloud.weave.works/k8s/net?&env.IPALLOC_RANGE=${CLUSTER_CIDR}&k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

> `CLUSTER_CIDR` must be inclined with the `--cluster-cidr` option when bootstraping Kubernetes controllers. Weave uses POD CIDR of `10.32.0.0/12` by default

Verify deployment:

```shell
kubectl --kubeconfig admin.kubeconfig get pods -n kube-system
```

Result:

```
NAME              READY   STATUS    RESTARTS        AGE
weave-net-6nvjd   2/2     Running   1 (2m20s ago)   3m2s
weave-net-nlrn5   2/2     Running   1 (2m19s ago)   3m2s
weave-net-pc74n   2/2     Running   1 (2m19s ago)   3m2s
```

Check weave status:

```shell
kubectl exec -n kube-system weave-net-6nvjd -c weave -- /home/weave/weave --local status
```

Next: [Deploying the DNS Cluster Add-on](12-dns-addon.md)
