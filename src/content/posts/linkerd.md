---
title: "Linkerd"
description: "meta description"
date: 2024-07-13T05:00:00Z
image: "/images/posts/01.jpg"
categories: ["kubernetes", "service mesh"]
authors: ["Balaji Balasundaram"]
tags: ["kubernetes", "k8s", "linkerd", "service mesh"]
draft: true
---

4. Verify Linkerd CLI Installation
Check the Linkerd CLI version to ensure it is installed correctly:

```bash
linkerd version
```
Step 2: Validate your Kubernetes cluster
```
linkerd check --pre
```

5. Install Linkerd on Minikube
Install the Linkerd control plane onto your Minikube cluster:

```bash
linkerd install --crds | kubectl apply -f -
#linkerd install | kubectl apply -f -
linkerd install --set proxyInit.runAsRoot=true | kubectl apply -f -
```
Check the installation:

```bash
linkerd check
```

6. Annotate Namespace for Proxy Injection
Annotate the namespace where your microservices are deployed to enable automatic proxy injection:

```bash
kubectl create namespace my-namespace
kubectl annotate namespace my-namespace linkerd.io/inject=enabled
```
Replace my-namespace with the name of your namespace.
