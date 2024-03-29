### **AUGUST 22, 2022 UPDATE**

After learning that i'd have to deal with k8s persistent volumes on this baremetal install, and of the potential issues resizing persistent volume claims, the kubernetes port of whitney is getting iceboxed and i'm going back to working on the docker deployment. See https://github.com/runyanjake/whitney for the current status of the whitney saga. Thanks!


# whitney-k8s
It's https://github.com/runyanjake/whitney, now with Kubernetes!

This will be initially built as a single node system, and will grow into an actual cluster as I source server hardware. For now it will serve as a more robust container management solution.

## Installation

#### Prereqs

Running on Ubuntu Server 22.04

Github CLI (optional, for downloading this repo easier)
https://www.techiediaries.com/install-github-cli-ubuntu-20/

#### Every Restart Notes

Sometimes swap is on again on restart. 
If the kubelet process (the daemon) gets stuck while running check your `/etc/fstab` and make sure the line about swap is removed so that the swap file isn't mounted. This needs to happen even if the `swapoff -a` is done, as fstab gets edited under the hood with swapoff.
After doing this, k8s should work on startup.

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

Untaint the master node - This is some weird step that you have to do for the master node to be allowed to work as a worker node. Otherwise there will be issues with the cluster.

```
kubectl taint nodes --all node-role.kubernetes.io/master-
```


##### Connect worker nodes

Use the kubeadm command saved from earlier to join worker bees into the swarm..  

Check for success with

```
kubectl get nodes
```

##### Remove Taint on Master Node

In order to schedule stuff on the master node, you must remove the taint on the master that disallows scheduling on the node (Can see this with `kubectl describe node master-node`.

`kubectl patch node master-node -p '{"spec":{"taints":[]}}'`
