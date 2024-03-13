
# Kubeadm Installation Guide

This guide outlines the steps needed to set up a Kubernetes cluster using kubeadm.


## Pre-requisites

- Ubuntu OS (Xenial or later)

- sudo privileges

- Internet access

- t2.medium instance type or higher
## Both Master & Worker Node

Run the following commands on both the master and worker nodes to prepare them for kubeadm.

```bash
# using 'sudo su' is not a good practice.
sudo apt update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo apt install docker.io -y

sudo systemctl enable --now docker # enable and start in single command.

# Adding GPG keys.
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add the repository to the sourcelist.
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update 
sudo apt install -y kubeadm=1.28.1-1.1 kubelet=1.28.1-1.1 kubectl=1.28.1-1.1 -y

```


## Master Node

1. Initialize the Kubernetes master node.

```bash
sudo kubeadm init

```

After succesfully running, your Kubernetes control plane will be initialized successfully.


2. Set up local kubeconfig (both for root user and normal user):

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

3. Apply Weave network:

```bash
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```
4. Generate a token for worker nodes to join:

```bash
sudo kubeadm token create --print-join-command
```

5. Expose port 6443 in the Security group for the Worker to connect to Master Node
## Worker Node

1. Run the following commands on the worker node.

```bash
sudo kubeadm reset pre-flight checks
```

2. Paste the join command you got from the master node and append ```--v=5 ```at the end. Make sure either you are working as ```sudo``` user or use sudo before the command
## Verify Cluster Connection

On Master Node:

```bash
kubectl get nodes
```
## Optional: Labeling Nodes

If you want to label worker nodes, you can use the following command:

```bash
kubectl label node <node-name> node-role.kubernetes.io/worker=worker
```
## Optional: Test a demo Pod

If you want to test a demo pod, you can use the following command:

```bash
kubectl run hello-world-pod --image=busybox --restart=Never --command -- sh -c "echo 'Hello, World' && sleep 3600"
```
