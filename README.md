# Kubernetes Vagrant

This repository includes steps for setting up Kubernetes on an Ubuntu machine using Vagrant.
I basically wanted my own development cluster, but had only invested in a single server (womp womp)!
To solve this problem.... virtual machines to the rescue! It is based on assets from [mbaykara/k8s-cluster](https://github.com/mbaykara/k8s-cluster) and [this article](https://medium.com/swlh/setup-own-kubernetes-cluster-via-virtualbox-99a82605bfcc).

## Steps

## 0. Clone the repository

```bash
$ git clone https://github.com/vsoch/k8s-vagrant
```

### 1. Install Dependencies

We need vagrant and virtual box
```bash
$ sudo apt-get update
$ sudo apt-get install -v vagrant virtualbox
```

## 2. Vagrant up

Then bring up the cluster!

```bash
$ vagrant up
```

## 3. Nodes

We will next want to ssh in to our main and worker nodes.

### main

```bash
$ vagrant ssh main
```
kubeadm ("Kube Admin") is installed, and you can use it to initialize the main node.

```bash
$ which kubeadm
/usr/bin/kubeadm
vagrant@main:~$ sudo kubeadm init --apiserver-advertise-address 192.168.33.13 --pod-network-cidr=10.244.0.0/16
[init] Using Kubernetes version: v1.23.1
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
...

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.33.13:6443 --token xxxxxxxxxxxxxxxxxxxx \
	--discovery-token-ca-cert-hash xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

When it is finished (you see the last line above) you'll want to setup your local kube config file.
If you've ever connected to an already running cluster, you likely used this:

```bash
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
You can try this, and see that nodes aren't ready:

```bash
$ kubectl get nodes
NAME   STATUS     ROLES                  AGE     VERSION
main   NotReady   control-plane,master   7m23s   v1.23.1
```

### workers

For each worker, do:

```bash
```
**under development**
