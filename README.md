# This is guide how to setup k8s cluster using kubeadm on VBox's VMs

## Prerequisites

## VM Hardware Requirements

8 GB of RAM (Preferebly 16 GB)
50 GB Disk space

## Virtual Box

Download and Install [VirtualBox](https://www.virtualbox.org/wiki/Downloads) on any one of the supported platforms:

 - Windows hosts
 - OS X hosts
 - Linux distributions
 - Solaris hosts

## Vagrant

Once VirtualBox is installed you may chose to deploy virtual machines manually on it.
Vagrant provides an easier way to deploy multiple virtual machines on VirtualBox more consistenlty.

Download and Install [Vagrant](https://www.vagrantup.com/) on your platform.

- Windows
- Debian
- Centos
- Linux
- macOS
- Arch Linux

## Step 0

Run next command from the vagrant directory

```shell script
$ vagrant status
```

Output:
```shell script
Current machine states:

kubemaster                not created (virtualbox)
kubenode01                not created (virtualbox)
```

## Step 1

```shell script
$ vagrant up
```

Command must be successfully finished, and status check should look like this:

```shell script
$ vagrant status
```

Output:
```shell script
Current machine states:

kubemaster                running (virtualbox)
kubenode01                running (virtualbox)
```

## Step 2

Test ssh connect to the newlly created nodes

```shell script
$ vagrant ssh kubemaster
```

```shell script
$ export kubernetesOperator="YOUR_NICKNAME_HERE"
```

```shell script
$ echo "Hello from kubeMaster! Dear $kubernetesOperator"
```

Do the same for the worker instance

```shell script
$ vagrant ssh kubenode01
```

```shell script
$ export kubernetesOperator="YOUR_NICKNAME_HERE"
```

```shell script
$ echo "Hello from kubeNode01! Dear $kubernetesOperator"
```