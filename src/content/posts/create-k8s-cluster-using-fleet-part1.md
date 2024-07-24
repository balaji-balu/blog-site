---
title: "Build K8s cluster using fleet (Part 1)"
description: "meta description"
date: 2024-07-14T05:00:00Z
image: "/images/posts/one.png"
categories: ["kubernetes"]
authors: ["Balaji Balasundaram"]
tags: ["fleet", "k8s", "kubernetes", "cluster"]
draft: false
---

# Build K8s cluster using fleet

Creating a three-node Kubernetes cluster using Rancher Fleet involves several steps. Below is a detailed guide to help you set up your cluster:

## Prerequisites
- **Rancher Server**: Ensure you have Rancher installed. You can install it on a VM or a physical server.
- **Fleet**: Rancher Fleet should be installed. Fleet is a GitOps tool that allows you to manage thousands of Kubernetes clusters.
- **Access to Three Nodes**: You should have three nodes (VMs or physical servers) ready to be configured as Kubernetes nodes.

## Steps to Create a 3-Node Kubernetes Cluster
### 1. Install Rancher
If you haven't already installed Rancher, follow the instructions to install Rancher on a single node. You can do this using Docker:

```sh
docker run -d --restart=unless-stopped \
    -p 80:80 -p 443:443 \
    --privileged \
    rancher/rancher:latest
```

### 2. Access Rancher UI
After installing Rancher, access the Rancher UI by navigating to https://<RANCHER_SERVER_IP>. Set up the initial admin password and URL.

### 3. Add Nodes to Rancher
In Rancher, go to the Cluster Management section and add a new custom cluster. Name your cluster and configure it as needed.

1. Click on Add Cluster.
2. Choose Custom.
3. Enter the cluster name and configure any necessary options.
4. Click on Next.

### 4. Prepare Nodes for Kubernetes
SSH into each of your three nodes and prepare them for Kubernetes installation:

```sh
sudo apt-get update
sudo apt-get install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
```

### 5. Register Nodes with Rancher
After preparing your nodes, use the command provided by Rancher to register each node. This command will look something like:

```sh
sudo docker run -d --privileged --restart=unless-stopped \
    --net=host \
    -v /etc/kubernetes:/etc/kubernetes \
    -v /var/run:/var/run \
    rancher/rancher-agent:v2.x.x \
    --server https://<RANCHER_SERVER_IP> \
    --token <REGISTRATION_TOKEN> \
    --ca-checksum <CA_CHECKSUM> \
    --etcd --controlplane --worker
```
Run this command on all three nodes. This will register the nodes with your Rancher server and set up the Kubernetes cluster.

Registration token can be generated using [kubeadm token](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-token/) or [k3s cli command](https://docs.k3s.io/cli/token#k3s-token-1)

### 6. Verify Cluster Setup
After running the command on all three nodes, go back to the Rancher UI. You should see the nodes being added to the cluster. Wait for the cluster status to become Active.

### 7. Install Fleet
Fleet should be installed by default with Rancher, but if not, you can install it manually.

1. Go to the Cluster Management section in Rancher.
2. Click on Apps & Marketplace.
3. Search for Fleet and install it.

### 8. Configure GitOps with Fleet
Fleet uses a Git repository to manage cluster configurations. You can create a Git repository and add a fleet.yaml file to define your cluster configuration.

Example fleet.yaml:

```yaml
defaultNamespace: fleet-local

helm:
  releaseName: my-app
  chart: ./charts/my-app
```
Push this configuration to your Git repository.

### 9. Add Git Repo to Fleet
1. In the Rancher UI, go to Cluster Management.
2. Click on Git Repos.
3. Click on Add Repo.
4. Enter the details of your Git repository.
5. Fleet will sync the repository and apply the configurations to your cluster.

## Summary
You have successfully created a three-node Kubernetes cluster using Rancher and configured it with Fleet for GitOps-based management. This setup allows you to manage your cluster configurations declaratively using Git.

#### References:
1. [Fleet](https://fleet.rancher.io/)
2. [Rancher Fleet: GitOps Across A Large Number Of Kubernetes Clusters](https://www.youtube.com/watch?v=rIH_2CUXmwM) DevOps Toolkit, Youtube
3. [Scaling Kubernetes Fleet Management Using GitOps Bridge](https://www.youtube.com/watch?v=QgcfvYmSbBw) CNCF, youtube
