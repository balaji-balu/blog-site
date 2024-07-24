---
title: "Building a Lightweight and Observable Kubernetes Cluster with K3s and Cilium"
description: "meta description"
date: 2022-04-01T05:00:00Z
image: "/images/posts/01.jpg"
categories: ["kubernetes"]
authors: ["Balaji Balasundaram"]
tags: ["cilium", "k3s", "kubernetes", "observability"]
draft: false
---

# Building a Lightweight and Observable Kubernetes Cluster with K3s and Cilium

In the modern cloud-native landscape, Kubernetes has become the de facto standard for container orchestration. However, the standard Kubernetes distribution can be quite resource-intensive, which might not be ideal for edge, IoT, or development environments. Enter K3s: a lightweight Kubernetes distribution built for these scenarios. Coupled with Cilium for enhanced networking and observability, you can achieve a robust, observable, and efficient Kubernetes setup. In this blog, we will explore how to set up K3s with Cilium and configure observability tools to monitor the cluster.

## What is K3s?

K3s is a lightweight Kubernetes distribution developed by Rancher Labs. It is designed to be a simple, easy-to-deploy, and fully conformant Kubernetes distribution that requires fewer resources and has a smaller footprint compared to the standard Kubernetes distribution.

### Key Features of K3s:

- **Lightweight:** Strips out legacy, alpha, and non-default features that are not required for most use cases.
- **Easy to Install:** A single binary less than 100 MB and a simple install script.
- **Reduced Dependencies:** Uses containerd instead of Docker, simplifies TLS management, and uses SQLite instead of etcd (although etcd is still supported).
- **Built-in Add-ons:** Comes with built-in add-ons like Helm, Traefik, and metrics-server.

## What is Cilium?

Cilium is an open-source networking, observability, and security solution for Kubernetes using eBPF (extended Berkeley Packet Filter). Cilium provides advanced networking features like:

- **Load Balancing:** Efficiently balances traffic across service endpoints.
- **Network Security:** Provides fine-grained network security with policies.
- **Observability:** Offers deep visibility into networking, providing insights into packet flows, policies, and performance.

## Setting Up K3s

### Prerequisites:

- A Linux-based system (e.g., Ubuntu 20.04).
- A user with `sudo` privileges.

### Installation Steps:

1. **Install K3s:**

   ```bash
   curl -sfL https://get.k3s.io | sh -

## Integrating Cilium with K3s

Prerequisites:
Helm installed on your system.
Installation Steps:
Install Cilium CLI:

```bash
curl -L --remote-name-all https://github.com/cilium/cilium-cli/releases/latest/download/cilium-linux-amd64.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-amd64.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-amd64.tar.gz /usr/local/bin
rm cilium-linux-amd64.tar.gz{,.sha256sum}
```
Install Cilium using Helm:

```bash
helm repo add cilium https://helm.cilium.io/
helm repo update
helm install cilium cilium/cilium --version 1.9.6 \
  --namespace kube-system \
  --set kubeProxyReplacement=strict \
  --set k8sServiceHost=127.0.0.1 \
  --set k8sServicePort=6443
```
Verify Cilium Installation:

```bash
cilium status
```

This command should show the status of Cilium components running in the cluster.

## Adding Observability
### Metrics Server:
K3s comes with a built-in metrics server which can be enabled during installation or later.

Install Metrics Server:

```bash
sudo k3s kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
Verify Metrics Server:

```bash
sudo k3s kubectl get apiservices | grep metrics
```
Prometheus and Grafana:
Prometheus and Grafana are popular tools for monitoring and observability in Kubernetes.

Add Helm Repositories:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```
Install Prometheus:

```bash
helm install prometheus prometheus-community/kube-prometheus-stack
```
Install Grafana:

```bash
helm install grafana grafana/grafana
```
Access Grafana:

Retrieve the Grafana admin password:

```bash
kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```
Forward the Grafana port to access it locally:

```bash
kubectl port-forward service/grafana 3000:80
```
Access Grafana at http://localhost:3000 and log in using the retrieved admin password.

Integrating Cilium Metrics:
Enable Cilium Metrics:

Edit the Cilium configuration to enable metrics collection:

```bash
helm upgrade cilium cilium/cilium --namespace kube-system --set prometheus.enabled=true
```
Add Cilium Dashboard to Grafana:

Import the Cilium dashboard into Grafana to visualize the networking metrics.

