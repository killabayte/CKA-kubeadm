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

# Letting iptables see bridged traffic
 
 Make sure  that the br_netfilter module is loaded. This can be done by running:

 ```shell script
 $ lsmod | grep br_netfilter
 ```

To load it explicitly call 

```shell script
$ sudo modprobe br_netfilter
```

As a requirement for your Linux Node's iptables to correctly see bridged traffic, you should ensure net.bridge.bridge-nf-call-iptables is set to 1 in your sysctl config, e.g.

```shell script
$ cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

$ sudo sysctl --system
```

# Install runtime. containerd

This section contains the necessary steps to use containerd as CRI runtime.
Use the following commands to install Containerd on your system:
Install and configure prerequisites:

```shell script
$ cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

$ sudo modprobe overlay
$ sudo modprobe br_netfilter

# Setup required sysctl params, these persist across reboots.
$ cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# Apply sysctl params without reboot
$ sudo sysctl --system
```

Install containerd:

```shell script
$ sudo apt-get update

$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

$ echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

$ sudo apt-get update

$ sudo apt-get install containerd.io
```

Configure containerd:

```shell script
$ sudo mkdir -p /etc/containerd
$ containerd config default | sudo tee /etc/containerd/config.toml
```

Restart containerd:

```shel script
$ sudo systemctl restart containerd
```

If you apply this change make sure to restart containerd again:

```shell script
$ sudo systemctl restart containerd
```

When using kubeadm, manually configure the [cgroup driver for kubelet](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#configure-cgroup-driver-used-by-kubelet-on-control-plane-node)

# Installing kubeadm, kubelet and kubectl

Update the apt package index and install packages needed to use the Kubernetes apt repository:

```shell script
$ sudo apt-get update
$ sudo apt-get install -y apt-transport-https ca-certificates curl
```

Download the Google Cloud public signing key:

```shell script
$ sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```

Add the Kubernetes apt repository:

```shell script
$ echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Update apt package index, install kubelet, kubeadm and kubectl, and pin their version:

```shell script
$ sudo apt-get update
$ sudo apt-get install -y kubelet kubeadm kubectl
$ sudo apt-mark hold kubelet kubeadm kubectl
```

# Creating a cluster with kubeadm

```shell script
$ sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=192.168.56.2
```
 **!Note: If you see error:** 

 ```shell script
 [init] Using Kubernetes version: v1.21.0
[preflight] Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR CRI]: container runtime is not running: output: time="2021-05-09T09:08:39Z" level=fatal msg="getting status of runtime failed: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.RuntimeService"
, error: exit status 1
 ```

 You can try to apply simple solution accrding to [this github issue](https://github.com/containerd/containerd/issues/4581) which should help to run kubeadm command successfully

 Output:
 
 ```shell script
 Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.56.2:6443 --token *****.*********** \
	--discovery-token-ca-cert-hash sha256:********************************************************
 ```

 Save this output for later and setup your kubeconfig

 ```shell script
 $ mkdir -p $HOME/.kube
 $ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 $ sudo chown $(id -u):$(id -g) $HOME/.kube/config
 ```

 Check if it works:

 ```shell script
 $ kubectl get nodes
 ```

 Output:

 ```shell script
 NAME         STATUS     ROLES                  AGE     VERSION
kubemaster   NotReady   control-plane,master   7m55s   v1.21.0
 ```

Now cluster requires network-plugin installation. Professional/Experts like us, choose [Calico](https://docs.projectcalico.org/getting-started/kubernetes/quickstart)

 # Install Calico

 Install the Tigera Calico operator and custom resource definitions.

```shell script
$ kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml
```

Install Calico by creating the necessary custom resource. For more information on configuration options available in this manifest, see the [installation reference](https://docs.projectcalico.org/reference/installation/api).

```shell script
$ kubectl create -f https://docs.projectcalico.org/manifests/custom-resources.yaml
```

Confirm that all of the pods are running with the following command.

```shell script
$ kubectl get pods -n calico-system -w
```

Wait until each pod has the STATUS of Running.

Check nodes status again:

```shell script
$ kubectl get nodes
```

Output:

```shell script
NAME         STATUS   ROLES                  AGE   VERSION
kubemaster   Ready    control-plane,master   13m   v1.21.0
```

Status must be READY

Otional. Enable on boot containerd and kubelet:

```shell script
$ sudo systemctl enable containerd
$ sudo systemctl enable kubelet
```