---
title: "Introduction to K3S: Lightweight Kubernetes for Edge and IoT"
description: "meta description"
date: 2024-07-14T05:00:00Z
image: "/images/posts/01.jpg"
categories: ["kubernetes"]
authors: ["Balaji Balasundaram"]
tags: ["kubernetes", "k3s", "cluster"]
draft: false
---

# An Introduction to K3S: Lightweight Kubernetes for Edge and IoT

Kubernetes has become the de facto standard for container orchestration, offering a robust and scalable platform for deploying, managing, and scaling containerized applications. However, the full Kubernetes distribution can be resource-intensive, making it less suitable for edge, IoT, and other resource-constrained environments. This is where K3S comes into play. Developed by Rancher Labs, K3S is a lightweight Kubernetes distribution designed to run on minimal hardware, providing a simplified and efficient solution for these scenarios.

## What is K3S?
K3S is a fully compliant Kubernetes distribution, certified by the Cloud Native Computing Foundation (CNCF). It is specifically designed to be lightweight, with a significantly reduced binary size and lower memory footprint compared to standard Kubernetes distributions. K3S is ideal for running on edge devices, IoT gateways, development environments, and other scenarios where resources are limited.

## Key Features of K3S
1. Lightweight and Efficient: K3S has a small binary size (less than 100 MB) and requires less memory, making it suitable for devices with limited resources.
2. Simplified Installation: The installation process for K3S is straightforward, with a single binary and minimal dependencies.
3. Embedded Components: K3S includes embedded components such as SQLite for storage, Traefik for ingress, CoreDNS for DNS management, and Flannel for networking.
4. Automatic TLS Management: K3S automatically manages TLS certificates, ensuring secure communication between cluster components.
5. ARM Support: K3S supports ARM architecture, making it possible to run on devices like Raspberry Pi and other ARM-based systems.
6. High Availability: K3S can be configured for high availability with multi-node clusters and supports external datastores.
7. Compatibility: Despite being lightweight, K3S maintains full compatibility with the Kubernetes API, allowing the use of standard Kubernetes tooling and APIs.

## Setting Up a K3S Cluster
Setting up a K3S cluster is straightforward and can be done in a few simple steps. Here’s how to get started:

### Prerequisites
- A Linux-based system (e.g., Ubuntu, CentOS) with a minimum of 512 MB of RAM and 2 CPUs.
- Access to the internet for downloading K3S and its dependencies.
- A user with sudo privileges.

### Step-by-Step Installation
#### 1. Install K3S on the Master Node:

```bash
curl -sfL https://get.k3s.io | sh -
```
This command downloads and installs K3S on your master node. The k3s service will start automatically, and you can check its status with:

```bash
sudo systemctl status k3s
```
#### 2. Get the Node Token:

To add worker nodes to your cluster, you need the node token. You can find it on the master node at:

```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```
#### 3. Install K3S on Worker Nodes:

On each worker node, run the following command, replacing <MASTER_IP> with the IP address of your master node and <NODE_TOKEN> with the token you retrieved:

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://<MASTER_IP>:6443 K3S_TOKEN=<NODE_TOKEN> sh -
```
This command joins the worker node to the K3S cluster.

#### 4. Verify the Cluster:

On the master node, use kubectl to verify that your nodes are up and running:

```bash
kubectl get nodes
```
You should see a list of all the nodes in your cluster.

### Maintaining a K3S Cluster
Maintaining a K3S cluster involves regular monitoring, updates, and backups to ensure smooth operation and minimal downtime.

#### Monitoring
Use tools like kubectl top to monitor resource usage:

```bash
kubectl top nodes
kubectl top pods
```
For more advanced monitoring, you can integrate Prometheus and Grafana with K3S.

#### Updating K3S
Updating K3S to the latest version ensures you have the latest features, bug fixes, and security patches. To update K3S, simply rerun the installation script on each node:

```bash
curl -sfL https://get.k3s.io | sh -
```
This will download and install the latest version of K3S, keeping your cluster up to date.

#### Backing Up K3S
Regular backups are essential for disaster recovery. You can back up the K3S data directory, which contains all the cluster state information. Use the following command to create a backup:

```bash
sudo tar -czvf k3s-backup.tar.gz /var/lib/rancher/k3s/server/db
```
Store the backup file in a secure location. To restore the cluster from a backup, extract the backup file to the original directory:

```bash
sudo tar -xzvf k3s-backup.tar.gz -C /
sudo systemctl restart k3s
```
### Conclusion
K3S offers a lightweight, efficient, and fully compliant Kubernetes distribution that is perfect for edge, IoT, and resource-constrained environments. With its simplified installation, embedded components, and easy maintenance, K3S makes it easier than ever to deploy and manage Kubernetes clusters in these scenarios. Whether you’re running on a Raspberry Pi or a large-scale edge deployment, K3S provides the power of Kubernetes with the simplicity and efficiency you need.


