 ## 1 - High availability Control Plan :
 ![[Pasted image 20231005024411.png]]
-Stacked etcd:
i should see mere about design petrons (stacked etcd, external etcd)

## 2- K8s Management tools:
- kubectl
- kubeadm (quickly create k8s clusters )
- Minikube (alow you use only on server for dev )
- Helm (provides templating and package management for k8s objects )
- kompose (Kompose helps you translate Docker compose files into Kubernetes objects. If you are using Docker compose for some part of your workflow, you can move your application to Kubernetes easily with Kompose.)
- Kustomize (it is semeler to Helm/allows you to share and re-use templated configurations for Kubernetes applications.)
## 3- Safely Draining a K8s Node
- What Is Draining?
>When performing maintenance, you may sometimes need to remove a Kubernetes node from service.

>To do this, you can drain the node. Containers running on the node will be gracefully terminated (and potentially rescheduled on another node).

```bash
kubectl drain <node name>
```

- Ignoring DaemonSets
>when draining nodes  there is some pods are tight to some specific nodes so we use this flag .

```bash
kubectl drain <node name> --ignore-daemonsets 
```

- Uncordoning a Node
>the node remains part of the cluster so when the maintenance completed we can  allow the pods run on the node.
```bash
kubectl uncordon <node name> 
```

## 4- Upgrading K8s with kubeadm
we need from time to time to upgrade our Kubernetes cluster.
so *kubeadm* make this process easy.
![[Pasted image 20231005040433.png]]
```bash
kubectl drain K8s-control --ignore-daemonsets 
```
```bash
# Upgrade kubeadm.

sudo apt-get update && \
sudo apt-get install -y --allow-change-held-packages kubeadm=1.27.2-00
kubeadm version

# Plan the upgrade.

sudo kubeadm upgrade plan v1.27.2
# Upgrade the control plane components.

sudo kubeadm upgrade apply v1.27.2

# Upgrade kubelet and kubectl on the control plane node.

sudo apt-get update && \
sudo apt-get install -y --allow-change-held-packages kubelet=1.27.2-00 kubectl=1.27.2-00

# Restart kubelet.

sudo systemctl daemon-reload
sudo systemctl restart kubelet

# Uncordon the control plane node.

kubectl uncordon k8s-control

# Verify that the control plane is working.

kubectl get nodes
```
###  Upgrade the worker nodes.
```bash
# Note: In a real-world scenario, you should not perform upgrades on all worker nodes at the same time. Make sure enough nodes are available at any given time to provide uninterrupted service.

# Run the following on the control plane node to drain worker node 1:

kubectl drain k8s-worker1 --ignore-daemonsets --force

# Log in to the first worker node, then Upgrade kubeadm.

sudo apt-get update && \
sudo apt-get install -y --allow-change-held-packages kubeadm=1.27.2-00

# Upgrade the kubelet configuration on the worker node.

sudo kubeadm upgrade node

# Upgrade kubelet and kubectl on the worker node.

sudo apt-get update && \
sudo apt-get install -y --allow-change-held-packages kubelet=1.27.2-00 kubectl=1.27.2-00

# Restart kubelet.

sudo systemctl daemon-reload
sudo systemctl restart kubelet

# From the control plane node, uncordon worker node 1.

kubectl uncordon k8s-worker1

# Repeat the upgrade process for worker node 2.
# From the control plane node, drain worker node 2.

kubectl drain k8s-worker2 --ignore-daemonsets --force

# On the second worker node, upgrade kubeadm.

sudo apt-get update && \
sudo apt-get install -y --allow-change-held-packages kubeadm=1.27.2-00

# Perform the upgrade on worker node 2.

sudo kubeadm upgrade node
sudo apt-get update && \
sudo apt-get install -y --allow-change-held-packages kubelet=1.27.2-00 kubectl=1.27.2-00
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# From the control plane node, uncordon worker node 2.

kubectl uncordon k8s-worker2

# Verify that the cluster is upgraded and working.

kubectl get nodes
```

## 5- Backing up and Restoring etcd
- Why back up etcd
>etcd is the backend data storage solution for your kubernetes cluster 

- we can Backing up etcd
```bash
ETCDCTL_API=3 etcdctl --endpoints $ENDPOINT snapshot save <file name>

```

- Restoring etcd 
```bash
ETCDCTL_API=3 etcdctl  snapshot restore <file name>

```