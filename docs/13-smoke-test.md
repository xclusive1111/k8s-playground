# Smoke Test

In this lab you will complete a series of tasks to ensure your Kubernetes cluster is functioning correctly.

## Data Encryption

In this section you will verify the ability to [encrypt secret data at rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#verifying-that-data-is-encrypted).

Create a generic secret:

```shell
kubectl create secret generic kubernetes-the-hard-way \
  --from-literal="mykey=mydata"
```

```
secret/kubernetes-the-hard-way created
```

Print a hexdump of the `kubernetes-the-hard-way` secret stored in etcd:

```
ssh master-0 "ETCDCTL_API=3 etcdctl get \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem\
  /registry/secrets/default/kubernetes-the-hard-way | hexdump -C"
```

> output

```
00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
00000010  73 2f 64 65 66 61 75 6c  74 2f 6b 75 62 65 72 6e  |s/default/kubern|
00000020  65 74 65 73 2d 74 68 65  2d 68 61 72 64 2d 77 61  |etes-the-hard-wa|
00000030  79 0a 6b 38 73 3a 65 6e  63 3a 61 65 73 63 62 63  |y.k8s:enc:aescbc|
00000040  3a 76 31 3a 6b 65 79 31  3a 23 ac d5 e8 e9 bb 63  |:v1:key1:#.....c|
00000050  ab c0 53 36 06 ca 14 e0  63 03 f2 ea 6a 21 07 91  |..S6....c...j!..|
00000060  96 9d 94 4b ea cd 83 f6  96 00 29 2f 92 d4 53 ff  |...K......)/..S.|
00000070  74 17 38 31 48 b3 78 8b  d7 c5 9f d8 26 c0 e6 f1  |t.81H.x.....&...|
00000080  09 5e 30 f9 e9 51 91 81  88 89 65 c3 7e 86 db 2c  |.^0..Q....e.~..,|
00000090  af 08 17 86 26 fe ce ad  2a 66 4e fe 70 7b 89 ee  |....&...*fN.p{..|
000000a0  1e 12 b6 bb 18 1e 05 ce  66 9b 03 b9 ea 9d d8 b8  |........f.......|
000000b0  cb 0c 08 f8 61 c0 f2 e7  c4 5c bb b2 fc 6f 3f 63  |....a....\...o?c|
000000c0  49 b5 fc 21 08 62 cf a3  d2 ce a2 bd bb 16 00 01  |I..!.b..........|
000000d0  65 7e cc a2 5d 40 68 12  ed 1e 03 40 05 41 4d b7  |e~..]@h....@.AM.|
000000e0  c2 c9 97 a2 ed 74 41 ed  11 f8 96 80 61 36 90 d2  |.....tA.....a6..|
000000f0  43 33 fb 29 b4 eb a0 95  93 f9 51 8f 23 6f e2 9b  |C3.)......Q.#o..|
00000100  27 4e bf f5 88 e6 2a 17  d5 7c 3e b9 37 ff 8b e8  |'N....*..|>.7...|
00000110  bd 26 6b 0d 0c 75 3f 59  ca 7b 77 f8 c1 7d 42 b7  |.&k..u?Y.{w..}B.|
00000120  25 62 58 70 89 c9 2f 8a  8a a5 9e 11 dc 4a 90 4a  |%bXp../......J.J|
00000130  89 6a 70 c3 c0 69 6e c5  3d 3d 84 6a e4 8b d5 30  |.jp..in.==.j...0|
00000140  1a 9b 38 b5 8d fb 1d 67  36 d2 25 e5 b4 00 f1 13  |..8....g6.%.....|
00000150  1b 81 e2 16 16 27 46 65  20 0a                    |.....'Fe .|
0000015a
```

The etcd key should be prefixed with `k8s:enc:aescbc:v1:key1`, which indicates the `aescbc` provider was used to encrypt the data with the `key1` encryption key.

## Deployments

In this section you will verify the ability to create and manage [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

Create a deployment for the [nginx](https://nginx.org/en/) web server:

```shell
kubectl create deployment nginx --image=nginx
```

List the pod created by the `nginx` deployment:

```shell
kubectl get pods -l app=nginx
```

> output

```
NAME                    READY   STATUS    RESTARTS   AGE
nginx-f89759699-kpn5m   1/1     Running   0          10s
```

### Port Forwarding

In this section you will verify the ability to access applications remotely using [port forwarding](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/).

Retrieve the full name of the `nginx` pod:

```shell
POD_NAME=$(kubectl get pods -l app=nginx -o jsonpath="{.items[0].metadata.name}")
```

Forward port `8080` on your local machine to port `80` of the `nginx` pod:

```
kubectl port-forward $POD_NAME 8080:80
```

```
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

In a new terminal make an HTTP request using the forwarding address:

```
curl --head http://127.0.0.1:8080
```

```
HTTP/1.1 200 OK
Server: nginx/1.21.6
Date: Wed, 01 Jun 2022 09:01:55 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 25 Jan 2022 15:03:52 GMT
Connection: keep-alive
ETag: "61f01158-267"
Accept-Ranges: bytes
```

Switch back to the previous terminal and stop the port forwarding to the `nginx` pod:

```
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
^C
```

### Logs

In this section you will verify the ability to [retrieve container logs](https://kubernetes.io/docs/concepts/cluster-administration/logging/).

Print the `nginx` pod logs:

```shell
kubectl logs $POD_NAME
```

```
...
127.0.0.1 - - [01/Jun/2022:09:01:55 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.83.1" "-"
```

### Exec

In this section you will verify the ability to [execute commands in a container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/#running-individual-commands-in-a-container).

Print the nginx version by executing the `nginx -v` command in the `nginx` container:

```shell
kubectl exec -ti $POD_NAME -- nginx -v
```

```
nginx version: nginx/1.21.6
```

## Services

In this section you will verify the ability to expose applications using a [Service](https://kubernetes.io/docs/concepts/services-networking/service/).

Expose the `nginx` deployment using a [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport) service:

```shell
kubectl expose deployment nginx --port 80 --type NodePort
```

> The LoadBalancer service type can not be used because your cluster is not configured with [cloud provider integration](https://kubernetes.io/docs/getting-started-guides/scratch/#cloud-provider). Setting up cloud provider integration is out of scope for this tutorial.

Retrieve the node port assigned to the `nginx` service:

```shell
NODE_PORT=$(kubectl get svc nginx \
  --output=jsonpath='{range .spec.ports[0]}{.nodePort}')
```

Retrieve the external IP address of a worker instance:

```shell
HOST_IP=$(kubectl get pods $POD_NAME -o wide --output=jsonpath='{.status.hostIP}')
```

Make an HTTP request using the host IP address and the `nginx` node port:

```shell
curl -I http://${HOST_IP}:${NODE_PORT}
```

> output

```
HTTP/1.1 200 OK
Server: nginx/1.21.6
Date: Wed, 01 Jun 2022 09:16:02 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 25 Jan 2022 15:03:52 GMT
Connection: keep-alive
ETag: "61f01158-267"
Accept-Ranges: bytes
```

Next: [Cleaning Up](14-cleanup.md)
