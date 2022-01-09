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

**todo going to try calico here instead**

Apply the networking policy:

```bash
$ kubectl apply -f https://raw.githubusercontent.com/vsoch/k8s-vagrant/main/k8s/network/kube-flannel.yml
Warning: policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
podsecuritypolicy.policy/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created
```
If you see nodes are unhealthy, follow instructions in [this post](https://medium.com/swlh/setup-own-kubernetes-cluster-via-virtualbox-99a82605bfcc). Mine were healthy :)

```bash
$ kubectl get cs
```

Then exit back to your local machine.

### workers

For each worker, do:

```bash
# Example to connect to each one
$ vagrant ssh worker-1
$ vagrant ssh worker-2
```

Note that this particular hash and token you'll need to copy from the main node output (the last line we saw above).
```bash
$ sudo kubeadm join 192.168.33.13:6443 --token xxxxxxxxxxxxxxxxxxxxxxxxxxx \
    --discovery-token-ca-cert-hash xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Then go back to the main node, and check your cluster again:

```bash
$ kubectl get nodes
NAME       STATUS   ROLES                  AGE     VERSION
main       Ready    control-plane,master   10m     v1.23.1
worker-1   Ready    <none>                 3m11s   v1.23.1
worker-2   Ready    <none>                 102s    v1.23.1
```

Now create a test deployment.

```bash
$ kubectl create deployment nginx --image=nginx --port 80
$ kubectl create deployment webserver --image=nginx --port 80 --replicas=5

To access from the internet we have to expose it as follow
$ kubectl expose deployment webserver --port 80 --type=NodePort
```

And then get the ip address:

```bash
$ kubectl describe services webserver
Name:                     webserver
Namespace:                default
Labels:                   app=webserver
Annotations:              <none>
Selector:                 app=webserver
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.1.240
IPs:                      10.96.1.240
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32502/TCP
Endpoints:                10.244.1.3:80,10.244.1.4:80,10.244.2.2:80 + 2 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```
And get it!

```bash
```
