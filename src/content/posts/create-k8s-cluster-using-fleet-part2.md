---
title: "Build K8s cluster using fleet (Part 2)"
description: "meta description"
date: 2024-07-14T05:00:00Z
image: "/images/posts/01.jpg"
categories: ["kubernetes"]
authors: ["Balaji Balasundaram"]
tags: ["fleet", "k8s", "kubernetes", "cluster"]
draft: false
---
# Build k8s cluster (part 2): add network and block storage to the cluster 
To add Cilium for networking and configure storage in your Kubernetes cluster using Rancher, follow the steps below:

## Prerequisites
- Kubernetes cluster with Rancher installed.
- Three nodes set up as described earlier.
- Ensure Helm is installed on your system for installing Cilium.

### Steps to Add Cilium for Networking
1. Install Helm (if not already installed):

```sh
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```
2. Add Cilium Helm Repository:

```sh
helm repo add cilium https://helm.cilium.io/
helm repo update
```
3. Install Cilium:

```sh
helm install cilium cilium/cilium --version 1.11.5 \
   --namespace kube-system \
   --set global.eni=false \
   --set global.etcd.enabled=false \
   --set global.etcd.managed=false \
   --set global.hubble.enabled=true \
   --set global.hubble.listenAddress=":4244" \
   --set global.hubble.relay.enabled=true \
   --set global.hubble.ui.enabled=true
```
This command installs Cilium with Hubble enabled for observability.

4. Verify Cilium Installation:

```sh
kubectl get pods -n kube-system -l k8s-app=cilium
```
Ensure all Cilium pods are running.

## Steps to Configure Storage (using Longhorn)
1. Install Longhorn:

In the Rancher UI, go to Cluster Management.
1. Select the cluster where you want to install Longhorn.
2. Navigate to Apps & Marketplace.
3. Search for Longhorn and click Install.
4. Follow the installation prompts and configure Longhorn as needed.

2. Verify Longhorn Installation:

```sh
kubectl get pods -n longhorn-system
```
Ensure all Longhorn pods are running.

3. Configure Storage Class:

Longhorn should automatically create a default StorageClass. Verify it using:

```sh
kubectl get storageclass
```
If you need to create a custom StorageClass, use the following manifest:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: longhorn
provisioner: driver.longhorn.io
reclaimPolicy: Retain
allowVolumeExpansion: true
volumeBindingMode: Immediate
```
Apply the manifest:

```sh
kubectl apply -f <custom-storageclass-manifest>.yaml
```
Verify Everything
Deploy a Test Application with Persistent Storage:
Create a YAML file for a test application, such as Nginx with a PersistentVolumeClaim (PVC):

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
  storageClassName: longhorn
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
          volumeMounts:
            - mountPath: "/usr/share/nginx/html"
              name: nginx-storage
      volumes:
        - name: nginx-storage
          persistentVolumeClaim:
            claimName: nginx-pvc
```
Apply the manifest:

```sh
kubectl apply -f <nginx-deployment>.yaml
```
Verify Application Deployment:

```sh
kubectl get pods
```
Ensure the Nginx pod is running and using the persistent storage provided by Longhorn.

### Summary
You have successfully added Cilium for networking and configured Longhorn for storage in your Kubernetes cluster managed by Rancher. This setup provides advanced networking capabilities with Cilium and reliable persistent storage with Longhorn.
