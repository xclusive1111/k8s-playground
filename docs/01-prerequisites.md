# Prerequisites

## VM Hardware Requirements

Preferably 16GB of RAM, 50GB of disk space.

This tutorial leverages the [LXC](https://linuxcontainers.org/lxc/getting-started/) to streamline provisioning of the
compute infrastructure required to bootstrap a Kubernetes cluster from the ground up.

## VirtualBox

Download and Install [VirtualBox](https://www.virtualbox.org/wiki/Downloads) on any one of the supported platforms:

- Windows hosts
- OS X hosts
- Linux distributions
- Solaris hosts

## Vagrant

Once VirtualBox is installed you may choose to deploy virtual machines manually on it.
Vagrant provides an easier way to deploy multiple virtual machines on VirtualBox more consistently.

Download and Install [Vagrant](https://www.vagrantup.com/downloads) on your platform.

- Windows
- Debian
- Centos
- Linux
- macOS

## Verification

`vagrant -v`:

> Output

```
2.2.19
```

## Running Commands in Parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time. Labs in this tutorial may require running the same commands across multiple compute instances, in those cases consider using tmux and splitting a window into multiple panes with synchronize-panes enabled to speed up the provisioning process.

> The use of tmux is optional and not required to complete this tutorial.

![tmux screenshot](images/tmux-screenshot.png)

> Enable synchronize-panes by pressing `ctrl+b` followed by `shift+:`. Next type `set synchronize-panes on` at the prompt. To disable synchronization: `set synchronize-panes off`.

Next: [Installing the Client Tools](02-client-tools.md)
