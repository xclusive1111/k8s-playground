# Provisioning Pod Network Routes

Pods scheduled to a node receive an IP address from the node's Pod CIDR range. At this point pods can not communicate with other pods running on different nodes due to missing network [routes](https://cloud.google.com/compute/docs/vpc/routes).

In this lab you will create a route for each worker node that maps the node's Pod CIDR range to the node's internal IP address.

> There are [other ways](https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-achieve-this) to implement the Kubernetes networking model.

## Deploy Calico Network

Run the following only once on the host that used to provision computing instances:

```shell
kubectl create -f https://projectcalico.docs.tigera.io/manifests/calico.yaml
```

> Calico will automatically detect the CIDR based on the running configuration

Verify deployment:

```shell
kubectl get pods -n kube-system -o wide
```

Result:

```
NAME                                         READY   STATUS    RESTARTS      AGE
calico-kube-controllers-56cdb7c587-qmzpf     1/1     Running   1 (22m ago)   32m
calico-node-hmdh5                            1/1     Running   1 (22m ago)   32m
calico-node-vzrv9                            1/1     Running   1 (22m ago)   32m
calico-node-xbtms                            1/1     Running   1 (22m ago)   32m
coredns-6cd56d4df4-6h4rw                     1/1     Running   7 (22m ago)   8d
coredns-6cd56d4df4-lm4j4                     1/1     Running   7 (22m ago)   8d

```



## Verify pods communication

Run two pods:

```shell
kubectl run box1 -it --rm --image busybox -- sh
```

```shell
kubectl run box2 -it --rm --image busybox -- sh
```

Check both pods are up and running:

```
NAME   READY   STATUS    RESTARTS   AGE   IP                NODE       NOMINATED NODE   READINESS GATES
box1   1/1     Running   0          74s   192.168.133.196   worker-2   <none>           <none>
box2   1/1     Running   0          70s   192.168.226.66    worker-1   <none>           <none>
```

> The IP addresses given to pods are managed by the chosen CNI IPAM plugin. Calico's IPAM plugin does not respect POD CIDR setup earlier, and instead manage it's own per-node blocks in order to provide better over IP address space usage.
> 
> You can use `kubectl get blockaffinities`to show the maping between node and CIDR that Calico is using.

In the shell of each box, ensure that they can `ping` each other.

Next: [Deploying the DNS Cluster Add-on](12-dns-addon.md)
