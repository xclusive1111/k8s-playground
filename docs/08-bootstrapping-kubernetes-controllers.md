# Bootstrapping the Kubernetes Control Plane

In this lab you will bootstrap the Kubernetes control plane across three compute instances and configure it for high availability. You will also create an external load balancer that exposes the Kubernetes API Servers to remote clients.

The following components will be installed on each node: Kubernetes API Server, Scheduler, and Controller Manager.

## Prerequisites

The commands in this lab must be run on each controller instance: `master-0`, `master-1`, and `master-2`.

Login to each controller instance using the `ssh` command. Example:

```shell
ssh master-0
```

### Running commands in parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time.
See the [Running commands in parallel with tmux](01-prerequisites.md#running-commands-in-parallel-with-tmux) section in
the Prerequisites lab.

## Provision the Kubernetes Control Plane

Create the Kubernetes configuration directory:

```shell
mkdir -p /etc/kubernetes/config
```

### Download and Install the Kubernetes Controller Binaries

Download the official Kubernetes release binaries:

```shell
wget -q --show-progress --https-only --timestamping \
  "https://storage.googleapis.com/kubernetes-release/release/v1.24.0/bin/linux/amd64/kube-apiserver" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.24.0/bin/linux/amd64/kube-controller-manager" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.24.0/bin/linux/amd64/kube-scheduler" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.24.0/bin/linux/amd64/kubectl"
```

Install the Kubernetes binaries:

```shell
{
  chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
  mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
}
```

### Configure the Kubernetes API Server

```shell
{
  mkdir -p /var/lib/kubernetes/
  mv ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
    service-account-key.pem service-account.pem \
    encryption-config.yaml /var/lib/kubernetes/
}
```

The instance internal IP address will be used to advertise the API Server to members of the cluster. Retrieve the internal IP address for the current compute instance:

```shell
INTERNAL_IP=$(ip -f inet addr show enp0s8 | sed -En -e 's/.*inet ([0-9.]+).*/\1/p')
```

Create the `kube-apiserver.service` systemd unit file:

```shell
KUBERNETES_PUBLIC_ADDRESS=$(cat /etc/hosts | grep lb-0 | awk '{print $1}')
MASTER_0=$(cat /etc/hosts | grep master-0 | awk '{print $1}')
MASTER_1=$(cat /etc/hosts | grep master-1 | awk '{print $1}')
MASTER_2=$(cat /etc/hosts | grep master-2 | awk '{print $1}')
MASTER_CNT=$(cat /etc/hosts | grep master | wc -l)
cat <<EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=${MASTER_CNT} \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=${MASTER_CNT} \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --enable-admission-plugins=NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota,ValidatingWebhookConfiguration \\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\
  --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \\
  --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \\
  --etcd-servers=https://${MASTER_0}:2379,https://${MASTER_1}:2379,https://${MASTER_2}:2379 \\
  --event-ttl=1h \\
  --encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
  --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \\
  --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \\
  --runtime-config='api/all=true' \\
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \\
  --service-account-signing-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-account-issuer=https://${KUBERNETES_PUBLIC_ADDRESS}:6443 \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \\
  --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \\
  --enable-aggregator-routing=true \\
  --proxy-client-key-file=/var/lib/kubernetes/kubernetes-key.pem \\
  --proxy-client-cert-file=/var/lib/kubernetes/kubernetes.pem \\
  --requestheader-client-ca-file=/var/lib/kubernetes/ca.pem \\
  --requestheader-allowed-names="kubernetes" \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Controller Manager

Move the `kube-controller-manager` kubeconfig into place:

```shell
mv kube-controller-manager.kubeconfig /var/lib/kubernetes/
```

Create the `kube-controller-manager.service` systemd unit file:

```shell
cat <<EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --bind-address=0.0.0.0 \\
  --cluster-cidr=10.200.0.0/16 \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \\
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.pem \\
  --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Scheduler

Move the `kube-scheduler` kubeconfig into place:

```shell
mv kube-scheduler.kubeconfig /var/lib/kubernetes/
```

Create the `kube-scheduler.yaml` configuration file:

```shell
cat <<EOF | sudo tee /etc/kubernetes/config/kube-scheduler.yaml
apiVersion: kubescheduler.config.k8s.io/v1beta3
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true
EOF
```

Create the `kube-scheduler.service` systemd unit file:

```shell
cat <<EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --config=/etc/kubernetes/config/kube-scheduler.yaml \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the Controller Services

```shell
{
  systemctl daemon-reload
  systemctl enable kube-apiserver kube-controller-manager kube-scheduler
  systemctl start kube-apiserver kube-controller-manager kube-scheduler
}
```

> Allow up to 10 seconds for the Kubernetes API Server to fully initialize.
> 
> Remember to run the above commands on each master node: `master-0`, `master-1`, and `master-2`.

### Kubernetes API Server Load Balancer

This section will boostrap an `haproxy` as a load balancer (layer 4) fronting the Kubernetes API Servers

Login into the `lb-0` instance:

```shell
ssh lb-0
```

Install `haproxy`:

```shell
apt-get install -y haproxy
```

Modify haproxy configuration:

```shell
{
MASTER_0=$(cat /etc/hosts | grep master-0 | awk '{print $1}')
MASTER_1=$(cat /etc/hosts | grep master-1 | awk '{print $1}')
MASTER_2=$(cat /etc/hosts | grep master-2 | awk '{print $1}')
cat <<EOF | tee -a /etc/haproxy/haproxy.cfg
frontend k8s
        bind 0.0.0.0:6443
        mode tcp
        default_backend k8s

backend k8s
        balance roundrobin
        mode tcp
        option tcplog
        option tcp-check
        server master-0 ${MASTER_0}:6443 check
        server master-1 ${MASTER_1}:6443 check
        server master-2 ${MASTER_2}:6443 check
EOF
}
```

Enable and start the haproxy service:

```shell
systemctl enable haproxy
systemctl restart haproxy
```

### Verification

Login into one of the master node:

```shell
ssh master-0
```

Check the cluster status:

```shell
kubectl cluster-info --kubeconfig admin.kubeconfig
```

Result:

```shell
Kubernetes control plane is running at https://127.0.0.1:6443
```

Test the API server version

```shell
LB=$(cat /etc/hosts | grep lb-0 | awk '{print $1}')
curl https://${LB}:6443/version \
--cacert /var/lib/kubernetes/ca.pem \
--key /var/lib/kubernetes/ca-key.pem
```

Result:

```json
{
  "major": "1",
  "minor": "24",
  "gitVersion": "v1.24.0",
  "gitCommit": "4ce5a8954017644c5420bae81d72b09b735c21f0",
  "gitTreeState": "clean",
  "buildDate": "2022-05-03T13:38:19Z",
  "goVersion": "go1.18.1",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

### 

## RBAC for Kubelet Authorization

In this section you will configure RBAC permissions to allow the Kubernetes API Server to access the Kubelet API on each
worker node. Access to the Kubelet API is required for retrieving metrics, logs, and executing commands in pods.

> This tutorial sets the Kubelet `--authorization-mode` flag to `Webhook`. Webhook mode uses the [SubjectAccessReview](https://kubernetes.io/docs/admin/authorization/#checking-api-access) API to determine authorization.

The commands in this section will affect the entire cluster and only need to be run once from one of the controller
nodes.

```shell
ssh master-0
```

Create the `system:kube-apiserver-to-kubelet` [ClusterRole](https://kubernetes.io/docs/admin/authorization/rbac/#role-and-clusterrole)
with permissions to access the Kubelet API and perform most common tasks associated with managing pods:

```shell
cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF
```

The Kubernetes API Server authenticates to the Kubelet as the `kubernetes` user using the client certificate as defined
by the `--kubelet-client-certificate` flag.

Bind the `system:kube-apiserver-to-kubelet` ClusterRole to the `kubernetes` user:

```shell
cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF
```

Next: [Bootstrapping the Kubernetes Worker Nodes](09-bootstrapping-kubernetes-workers.md)
