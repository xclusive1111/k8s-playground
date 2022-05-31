# Provisioning Compute Resources

Clone this repository and execute `vagrant up` at the root directory. This does the below:

- Deploys 7 VMs: 3 masters, 3 workers and 1 load balancer with the name `kube-*`
  
  > This is the default settings. This can be changed at the top of the Vagrant file.

- Set's IP addresses in the range 192.168.33.X
  
  | VM       | VM Name       | Purpose      | IP            | Forwarded Port |
  | -------- | ------------- | ------------ | ------------- | -------------- |
  | master-1 | kube-master-1 | Master       | 192.168.33.10 | 3210           |
  | master-2 | kube-master-2 | Master       | 192.168.33.11 | 3111           |
  | master-3 | kube-master-2 | Master       | 192.168.33.12 | 3212           |
  | worker-1 | kube-worker-1 | Worker       | 192.168.33.20 | 3220           |
  | worker-2 | kube-worker-2 | Worker       | 192.168.33.21 | 3221           |
  | worker-3 | kube-worker-2 | Worker       | 192.168.33.22 | 3222           |
  | lb-0     | kube-lb       | LoadBalancer | 192.168.33.30 | 3230           |

## SSH to the nodes

There are two ways to SSH into the nodes:

### 1. SSH using Vagrant

From the directory you ran the `vagrant up` command, run `vagrant ssh <vm>`, e.g: `vagrant ssh master-1`.

> Note: Use VM field from the above table and not the vm name itself.

### 2. SSH Using SSH Client Tools

Use your favourite SSH terminal tool. 

Use the above IP addresses. Username and password based SSH is disabled by default. Vagrant generates a private key for each of these VMs. It is placed under the `.vagrant` folder (in the directory you ran the `vagrant up` command from) at the below path for each VM:

- **Private key path:** `.vagrant/machines/<vm>/virtualbox/private_key`
- **Username:** `vagrant`

### 3. SSH from the host

> This is the recommended approach, and we will be using this to provision nodes in the next labs

- Make sure you update the host entries to include nodes' IPs:
  
  For Linux, the host file resign under `/etc/hosts`:
  
  ```
  192.168.33.10 master-0
  192.168.33.11 master-1
  192.168.33.12 master-2
  
  192.168.33.20 worker-0
  192.168.33.21 worker-1
  192.168.33.22 worker-2
  
  192.168.33.30 lb-0
  ```

- Login into each node as a `root` user and add your SSH key

- Create SSH profile for each node:
  
  ```
  Host master-0
      HostName 192.168.33.10
      User root
      IdentityFile ~/.ssh/id_rsa
  Host master-1
      HostName 192.168.33.11
      User root
      IdentityFile ~/.ssh/id_rsa
  Host master-2
      HostName 192.168.33.12
      User root
      IdentityFile ~/.ssh/id_rsa
  Host worker-0
      HostName 192.168.33.20
      User root
      IdentityFile ~/.ssh/id_rsa
  Host worker-1
      HostName 192.168.33.21
      User root
      IdentityFile ~/.ssh/id_rsa
  Host worker-2
      HostName 192.168.33.22
      User root
      IdentityFile ~/.ssh/id_rsa
  Host lb-0
      HostName 192.168.33.30
      User root
      IdentityFile ~/.ssh/id_rsa
  ```

## Verify Environment

- Ensure all VMs are up
- Ensure VMs are assigned the above IP addresses
- Ensure you can SSH into these VMs using SSH profile
- Ensure the VMs can ping each other

Useful commands:

- `vagrant up <vm>` to start up a VIM
- `vagrant halt <vm>` to shut down a VM gracefully
- `vagrant destroy <vm>` to completely remove a VM

Next: [Provisioning a CA and Generating TLS Certificates](04-certificate-authority.md)
