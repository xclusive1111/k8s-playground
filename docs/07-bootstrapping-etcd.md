# Bootstrapping the etcd Cluster

Kubernetes components are stateless and store cluster state in [etcd](https://github.com/etcd-io/etcd). In this lab you
will bootstrap a three node etcd cluster and configure it for high availability and secure remote access.

## Prerequisites

The commands in this lab must be run on each controller instance: `master-0`, `master-1`, and `master-2`.
Login to each master instance using the `ssh` command. Example:

```shell
ssh master-0
```

### Running commands in parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time.
See the [Running commands in parallel with tmux](01-prerequisites.md#running-commands-in-parallel-with-tmux) section in
the Prerequisites lab.

## Bootstrapping an etcd Cluster Member

### Download and Install the etcd Binaries

Download the official etcd release binaries from the [etcd](https://github.com/etcd-io/etcd) GitHub project:

```shell
wget -q --show-progress --https-only --timestamping \
  "https://github.com/etcd-io/etcd/releases/download/v3.5.4/etcd-v3.5.4-linux-amd64.tar.gz"
```

Extract and install the `etcd` server and the `etcdctl` command line utility:

```shell
{
  tar -xvf etcd-v3.5.4-linux-amd64.tar.gz
  mv etcd-v3.5.4-linux-amd64/etcd* /usr/local/bin/
  rm -r etcd-v3.5.4-linux-amd64 etcd-v3.5.4-linux-amd64.tar.gz
}
```

### Configure the etcd Server

```shell
{
  mkdir -p /etc/etcd /var/lib/etcd
  chmod 700 /var/lib/etcd
  cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
}
```

Export the internal IP address:

```shell
INTERNAL_IP=$(ip -f inet addr show enp0s8 | sed -En -e 's/.*inet ([0-9.]+).*/\1/p')
```

Each etcd member must have a unique name within an etcd cluster. Set the etcd name to match the hostname of the current
compute instance:

```shell
ETCD_NAME=$(hostname -s)
```

Create the `etcd.service` systemd unit file:

```shell
MASTER_0=$(cat /etc/hosts | grep master-0 | awk '{print $1}')
MASTER_1=$(cat /etc/hosts | grep master-1 | awk '{print $1}')
MASTER_2=$(cat /etc/hosts | grep master-2 | awk '{print $1}')
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster master-0=https://${MASTER_0}:2380,master-1=https://${MASTER_1}:2380,master-2=https://${MASTER_2}:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the etcd Server

```shell
{
  systemctl daemon-reload
  systemctl enable etcd
  systemctl start etcd
}
```

> Remember to run the above commands on each controller node: `master-0`, `master-1`, and `master-2`.

## Verification

List the etcd cluster members:

```shell
ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
```

```
12686a56262ba9c6, started, master-1, https://192.168.33.11:2380, https://192.168.33.11:2379, false
343245731c8534ff, started, master-2, https://192.168.33.12:2380, https://192.168.33.12:2379, false
7473d5f4df8d1b3c, started, master-0, https://192.168.33.10:2380, https://192.168.33.10:2379, false
```

Next: [Bootstrapping the Kubernetes Control Plane](08-bootstrapping-kubernetes-controllers.md)
