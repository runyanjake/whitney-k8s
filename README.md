# whitney-k8s
It's https://github.com/runyanjake/whitney, now with Kubernetes!

This will be initially built as a single node system, and will grow into an actual cluster as I source server hardware. For now it will serve as a more robust container management solution.

### Installation

0. Prereqs

- I'm running Ubuntu 20.04 on the original whitney machine but will switch to a Debian distro for the new hardware.

1. Prep Steps 
- https://phoenixnap.com/kb/how-to-install-kubernetes-on-a-bare-metal-server

...Disable swap, k8s may thrash disk and thus won't run if swap enabled.
`sudo swapoff -a`

...Set iptables tooling to use legacy mode because kubeadm cmds don't work with nftables.
```
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
sudo update-alternatives --set arptables /usr/sbin/arptables-legacy
sudo update-alternatives --set ebtables /usr/sbin/ebtables-legacy
```

...Open the required ports on the firewall on master node.
```
sudo ufw allow 6443/tcp
sudo ufw allow 2379/tcp
sudo ufw allow 2380/tcp
sudo ufw allow 10250/tcp
sudo ufw allow 10251/tcp
sudo ufw allow 10252/tcp
sudo ufw allow 10255/tcp
sudo ufw reload
```

...and on the worker nodes although i only have the master node for now.
```
sudo ufw allow 10251/tcp
sudo ufw allow 10255/tcp
sudo ufw reload
```

2. Install & Configure Docker

...Install docker.
```
which docker
```

...Change the cgroup-driver..  
```
sudo docker info | grep -i cgroup
```
...Confirm the response is `Cgroup Driver: cgroupfs`..  
...If not change it with
```
cat > /etc/docker/daemon.json <<EOF
{
"exec-opts": ["native.cgroupdriver=systemd"],
"log-driver": "json-file",
"log-opts": {
"max-size": "100m"
},
"storage-driver": "overlay2"
}
EOF
```
```
systemctl daemon-reload
```
```
systemctl restart kubelet
```

3. Install Kubernetes

...Check availability
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add –
```

...Add Xenial repo for k8s.
```
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

..Install Kubelet, Kubeadm, Kubectl
```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
...After the last command, kubelet will go into a reboot loop as it waits for instructions from Kubeadm.

...Initialize Kubernetes on master node
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

...Wait until confirmation that k8s has started successfully. SAVE THE KUBEADM JOIN MESSAGE FOR FUTURE USE

4. Configure kubectl for non-root users
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

5. Install a Pod network addon

...Necessary for pods to communicate effectively. The doc recommends Flannel but i'll use Weave Net..  
...`https://www.weave.works/docs/net/latest/kubernetes/kube-addon/`
```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

6. Connect worker nodes

...Use the kubeadm command saved from earlier to join worker bees into the swarm..  
...Check for success with
```
kubectl get nodes
```
