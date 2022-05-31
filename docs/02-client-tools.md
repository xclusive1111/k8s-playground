# Installing the Client Tools

In this lab you will install the command line utilities required to complete this tutorial:  [cfssl](https://github.com/cloudflare/cfssl), [cfssljson](https://github.com/cloudflare/cfssl), and [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl).

## Install CFSSL

The `cfssl` and `cfssljson` command line utilities will be used to provision a [PKI Infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure) and generate TLS certificates.

Download and install `cfssl` and `cfssljson`:

```shell
wget -c https://github.com/cloudflare/cfssl/releases/download/v1.6.1/cfssl_1.6.1_linux_amd64 -O cfssl
wget -c https://github.com/cloudflare/cfssl/releases/download/v1.6.1/cfssljson_1.6.1_linux_amd64 -O cfssljson
```

```shell
chmod +x cfssl cfssljson
```

```shell
sudo mv cfssl cfssljson /usr/local/bin/
```

### Verification

Verify `cfssl` and `cfssljson` version 1.6.1 or higher is installed:

`cfssl version`:

```
Version: 1.6.1
Runtime: go1.12.12
```

`cfssljson --version`:

```
Version: 1.6.1
Runtime: go1.12.12
```

## Install kubectl

The `kubectl` command line utility is used to interact with the Kubernetes API Server. Download and install `kubectl` from the official release binaries:

```shell
curl -LO "https://dl.k8s.io/release/v1.24.0/bin/linux/amd64/kubectl"
```

```shell
chmod +x kubectl
```

```shell
sudo mv kubectl /usr/local/bin/
```

### Verification

Verify `kubectl` version 1.24.0 or higher is installed:

`kubectl version --client`:

```
Client Version: version.Info{Major:"1", Minor:"24", GitVersion:"v1.24.0", GitCommit:"4ce5a8954017644c5420bae81d72b09b735c21f0", GitTreeState:"clean", BuildDate:"2022-05-03T13:46:05Z", GoVersion:"go1.18.1", Compiler:"gc", Platform:"linux/amd64"}
```

Next: [Provisioning Compute Resources](03-compute-resources.md)
