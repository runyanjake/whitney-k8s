# whitney-k8s
It's https://github.com/runyanjake/whitney, now with Kubernetes!

This will be initially built as a single node system, and will grow into an actual cluster as I source server hardware. For now it will serve as a more robust container management solution.

## Installation

#### Prereqs

Running on Ubuntu Server 22.04

#### Docker and K8s Installation Steps 

https://www.letscloud.io/community/how-to-install-kubernetesk8s-and-docker-on-ubuntu-2004

#### K8s Setup Steps

##### Once everything is installed, initialize the master node.

```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

Wait until confirmation that k8s has started successfully. SAVE THE KUBEADM JOIN MESSAGE FOR FUTURE USE, each worker node will have to use it to connect to the master node.

##### File system setup for a non-root user.

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

##### Install a Pod network addon

Necessary for pods to communicate effectively. The doc recommends Flannel but i'll use Weave Net..  

`https://www.weave.works/docs/net/latest/kubernetes/kube-addon/`

```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

##### Connect worker nodes

Use the kubeadm command saved from earlier to join worker bees into the swarm..  

Check for success with

```
kubectl get nodes
```
