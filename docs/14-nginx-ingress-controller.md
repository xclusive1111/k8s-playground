# Nginx Ingress Controller

In this lab, we will install an nginx ingress controller in order for the Ingress resource to work.

### Install the nginx ingress controller

Run this command on the host machine that used to provision compute resources:

```shell
kubectl apply -f ./deployment/ingress-controller.yaml
```

```
amespace/ingress-nginx created
serviceaccount/ingress-nginx created
serviceaccount/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
configmap/ingress-nginx-controller created
service/ingress-nginx-controller created
service/ingress-nginx-controller-admission created
deployment.apps/ingress-nginx-controller created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
ingressclass.networking.k8s.io/nginx created
```

Wait until ingress controller is ready:

```shell
 kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s
```

```
pod/ingress-nginx-controller-2whlr condition met
pod/ingress-nginx-controller-l7clv condition met
pod/ingress-nginx-controller-p5lbw condition met
```

> References:
> 
> - [Installation Guide - NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal-clusters)
> 
> - [Bare-metal considerations - NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/deploy/baremetal/#using-a-self-provisioned-edge)

### Verify ingress controller was installed properly

Checking ingress controller version:

```shell
POD_NAMESPACE=ingress-nginx
POD_NAME=$(kubectl get pods -n $POD_NAMESPACE -l app.kubernetes.io/name=ingress-nginx --field-selector=status.phase=Running -o name | head -n 1)
kubectl exec $POD_NAME -n $POD_NAMESPACE -- /nginx-ingress-controller --version
```

```
-------------------------------------------------------------------------------
NGINX Ingress controller
  Release:       v1.2.0
  Build:         a2514768cd282c41f39ab06bda17efefc4bd233a
  Repository:    https://github.com/kubernetes/ingress-nginx
  nginx version: nginx/1.19.10

-------------------------------------------------------------------------------
```

A DaemonSet is deployed that will schedules exactly one Pod on each worker node, and bind ports 80 & 443 directly on those nodes:

```shell
for instance in worker-0 worker-1 worker-2; do
    curl -I http://${instance}
done
```

```
HTTP/1.1 404 Not Found
Date: Fri, 03 Jun 2022 06:42:37 GMT
Content-Type: text/html
Content-Length: 146
Connection: keep-alive

HTTP/1.1 404 Not Found
Date: Fri, 03 Jun 2022 06:42:37 GMT
Content-Type: text/html
Content-Length: 146
Connection: keep-alive

HTTP/1.1 404 Not Found
Date: Fri, 03 Jun 2022 06:42:38 GMT
Content-Type: text/html
Content-Length: 146
Connection: keep-alive
```

Test creating ingress resource:

```shell
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test
spec:
  rules:
  - http:
      paths:
      - path: /test
        pathType: ImplementationSpecific
        backend:
          service:
            name: test
            port:
              number: 8080
EOF
```

```shell
ingress.networking.k8s.io/test created
```

Cleanup the tested ingress resource:

```shell
kubectl delete ingress test
```

```
ingress.networking.k8s.io "test" deleted
```

### Load balancer for the ingress controller

On a bare-metal Kubernetes installation, external clients do not access cluster nodes directly. We need a dedicated public IP address that forwards all HTTP traffic to Kubernetes worker nodes. Incoming traffic on TCP ports 80 and 443 is forwarded to the corresponding HTTP and HTTPS NodePort on the worker nodes.

We're going to modify haproxy configuration forward all HTTPS traffic to worker nodes on port 443 (layer 4 load balancer).

Run the following command on the `lb-0` instance:

```shell
{
WORKER_0=$(cat /etc/hosts | grep worker-0 | awk '{print $1}')
WORKER_1=$(cat /etc/hosts | grep worker-1 | awk '{print $1}')
WORKER_2=$(cat /etc/hosts | grep worker-2 | awk '{print $1}')
cat <<EOF | tee -a /etc/haproxy/haproxy.cfg

frontend k8s-public
        bind 0.0.0.0:443
        mode tcp
        default_backend k8s-public

backend k8s-public
        balance roundrobin
        mode tcp
        option tcplog
        option tcp-check
        server worker-0 ${WORKER_0}:443 check
        server worker-1 ${WORKER_1}:443 check
        server worker-2 ${WORKER_2}:443 check
EOF
}
```

Restart the haproxy systemd service

```shell
systemctl restart haproxy
```

Verify:

```shell
curl -I https://lb-0 -k
```

```
HTTP/2 404 
date: Fri, 03 Jun 2022 08:04:12 GMT
content-type: text/html
content-length: 146
strict-transport-security: max-age=15724800; includeSubDomains
```

Deploy sample services:

```shell
kubectl apply -f ./deployments/web1.yaml ./deployments/web2.yaml
```

Verify ingress deployment:

```shell
curl https://lb-0/v1 -k
```

```
Hello, world!
Version: 1.0.0
Hostname: web1-5d44788485-wlnzd
```

```shell
curl https://lb-0/v2 -k
```

```
Hello, world!
Version: 2.0.0
Hostname: web2-574fc47679-gnqlf
```
