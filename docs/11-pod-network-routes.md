# Provisioning Pod Network Routes

Pods scheduled to a node receive an IP address from the node's Pod CIDR range. At this point pods can not communicate with other pods running on different nodes due to missing network [routes](https://cloud.google.com/compute/docs/vpc/routes).

In this lab you will create a route for each worker node that maps the node's Pod CIDR range to the node's internal IP address.

> There are [other ways](https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-achieve-this) to implement the Kubernetes networking model.

## Deploy Calico Network

Run the following only once on the `master` node:

```shell
kubectl --kubeconfig admin.kubeconfig create -f https://projectcalico.docs.tigera.io/manifests/tigera-operator.yaml
```

> Calico will automatically detect the CIDR based on the running configuration

Verify deployment:

```shell
kubectl get pods --all-namespaces -o wide
```

Result:

```
NAMESPACE          NAME                                       READY   STATUS    RESTARTS   AGE   IP                NODE       NOMINATED NODE   READINESS GATES
calico-apiserver   calico-apiserver-6458f949b-59xbv           1/1     Running   0          22m   192.168.43.1      worker-0   <none>           <none>
calico-apiserver   calico-apiserver-6458f949b-pxczx           1/1     Running   0          22m   192.168.133.193   worker-2   <none>           <none>
calico-system      calico-kube-controllers-69cc7b7f48-gspww   1/1     Running   0          29m   10.200.1.4        worker-1   <none>           <none>
calico-system      calico-node-2524g                          1/1     Running   0          29m   192.168.33.22     worker-2   <none>           <none>
calico-system      calico-node-8nzd7                          1/1     Running   0          29m   192.168.33.21     worker-1   <none>           <none>
calico-system      calico-node-hml7l                          1/1     Running   0          29m   192.168.33.20     worker-0   <none>           <none>
calico-system      calico-typha-6ff7d59b7f-jcgmb              1/1     Running   0          29m   192.168.33.22     worker-2   <none>           <none>
calico-system      calico-typha-6ff7d59b7f-wrn9j              1/1     Running   0          29m   192.168.33.21     worker-1   <none>           <none>
tigera-operator    tigera-operator-5fb55776df-8xlgw           1/1     Running   0          30m   192.168.33.20     worker-0   <none>           <none>
```

> It can take up to 10 minutes to get all pods up and running.

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
