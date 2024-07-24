---
title: "Create baremetal kubernetes(k8s) multi-clusters using cluster api"
description: "meta description"
date: 2024-07-13T05:00:00Z
image: "/images/posts/01.jpg"
categories: ["kubernetes"]
authors: ["Balaji Balasundaram"]
tags: ["kubernetes", "k8s", "multi-cluster", "k3s", "cluster api"]
draft: true
---

- [DIY: Create Your Own Cloud with Kubernetes (Part 1)](https://kubernetes.io/blog/2024/04/05/diy-create-your-own-cloud-with-kubernetes-part-1/)
- [DIY: Create Your Own Cloud with Kubernetes (Part 2)](https://kubernetes.io/blog/2024/04/05/diy-create-your-own-cloud-with-kubernetes-part-2/)
- [DIY: Create Your Own Cloud with Kubernetes (Part 3)](https://kubernetes.io/blog/2024/04/05/diy-create-your-own-cloud-with-kubernetes-part-3/)

# Build kubernetes multi-cluster
 
Introduction goes here.

## Kubernetes (K8s) Cluster API
The Kubernetes Cluster API (Cluster API) is a Kubernetes project designed to manage the lifecycle of Kubernetes clusters. It provides declarative APIs and tooling to create, configure, and manage Kubernetes clusters in a consistent and automated manner. The Cluster API abstracts the infrastructure-specific details, enabling consistent cluster operations across different cloud providers and environments.

Within Cluster API, there are several types of providers. The major ones are:

- **Infrastructure Provider**, which is responsible for providing the computing infrastructure, such as virtual machines or physical servers. To run Kubernetes clusters using `KubeVirt`, the `KubeVirt Infrastructure Provider` must be installed.

- **Control Plane Provider**, which provides the Kubernetes control plane, namely the components kube-apiserver, kube-scheduler, and kube-controller-manager. The Kamaji project offers a ready solution for running the Kubernetes control plane for tenant clusters as containers within the management cluster.

- **Bootstrap Provider**, which is used for generating cloud-init configuration for the virtual machines and servers being created. `Kubeadm` as the Bootstrap Provider - as the standard method for preparing clusters in Cluster API.

## Gitops
.

### Key Features of K3s:

.

## Storage 

## Networking 

## using rancher fleet 

https://fleet.rancher.io/