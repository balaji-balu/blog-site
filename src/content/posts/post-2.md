---
title: Deploying K3s on Edge Devices for IoT Applications
description: "meta description"
date: 2024-07-08T05:00:00Z
image: "/images/posts/02.jpg"
categories: ["k3s edge"]
authors: ["Balaji Balasundaram"]
tags: ["edge", "k3s", "iot"]
draft: false
---
# Deploying K3s on Edge Devices for IoT Applications

## Introduction

The Internet of Things (IoT) has revolutionized industries by enabling real-time data collection, analysis, and automation. However, managing a large number of IoT devices can be challenging, especially in terms of scalability, reliability, and efficiency. Kubernetes has become a standard for orchestrating containerized applications, but its complexity and resource requirements make it less suitable for edge devices. Enter K3s, a lightweight Kubernetes distribution designed for resource-constrained environments. This article explores the deployment of K3s on edge devices for IoT applications, highlighting its benefits, architecture, and practical implementation.

## Why K3s for Edge and IoT?

K3s is a certified Kubernetes distribution by Rancher Labs that is optimized for resource-constrained environments. It is particularly suited for edge computing and IoT for several reasons:

1. **Lightweight**: K3s reduces the memory and CPU requirements by removing unnecessary features and components, making it ideal for edge devices.
2. **Simplicity**: The installation and management of K3s are straightforward, reducing the operational overhead.
3. **Embedded Components**: K3s integrates SQLite as the default datastore, although it can also use etcd, MySQL, or Postgres.
4. **Optimized for ARM**: K3s supports ARM64 and ARMv7, making it compatible with a wide range of edge devices like Raspberry Pi.

## Architecture Overview

The architecture of a typical K3s deployment on edge devices consists of the following components:

1. **K3s Server**: The server node that runs the Kubernetes control plane. It manages the state of the cluster, schedules applications, and handles API requests.
2. **K3s Agent**: The agent nodes that run the actual workloads. They communicate with the server node and execute the containerized applications.

A common deployment pattern involves a central cloud-based K3s server managing multiple edge-based K3s agents.

## Installation and Setup

### Prerequisites

- Edge device (e.g., Raspberry Pi) with a 64-bit operating system.
- Basic knowledge of Linux command-line operations.
- Network connectivity between the edge devices and the central K3s server.

### Installing K3s on the Server Node

First, we install K3s on the server node. The server node can be a powerful machine in a data center or cloud environment.

```sh
curl -sfL https://get.k3s.io | sh -
```
This command installs K3s and starts the server. You can verify the installation with:

```sh
sudo k3s kubectl get nodes
```

### Installing K3s on the Agent Nodes
Next, we install K3s on the edge devices, which act as agent nodes. Replace <SERVER_IP> and <TOKEN> with the appropriate values from your K3s server.

```sh
curl -sfL https://get.k3s.io | K3S_URL=https://<SERVER_IP>:6443 K3S_TOKEN=<TOKEN> sh -
```
Verify the agent node is connected to the server:

```sh
sudo k3s kubectl get nodes
```
You should see both the server and agent nodes listed.

## Deploying IoT Applications
With K3s up and running, you can deploy your IoT applications using Kubernetes manifests. Here is an example of deploying a simple IoT sensor application:

### IoT Sensor Application Manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iot-sensor
spec:
  replicas: 2
  selector:
    matchLabels:
      app: iot-sensor
  template:
    metadata:
      labels:
        app: iot-sensor
    spec:
      containers:
      - name: sensor
        image: your-iot-sensor-image:latest
        ports:
        - containerPort: 8080
```

Apply the manifest using `kubectl`:

```sh
sudo k3s kubectl apply -f iot-sensor.yaml
```

### Monitoring and Management
You can monitor and manage your IoT applications using standard Kubernetes tools like kubectl, as well as integrate with tools like Prometheus and Grafana for more advanced monitoring.
