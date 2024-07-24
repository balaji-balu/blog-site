---
title: "Building a K8s webapp using postgres running on Longhorn"
description: "meta description"
date: 2022-04-01T05:00:00Z
image: "/images/posts/two.png"
categories: ["kubernetes"]
authors: ["Balaji Balasundaram"]
tags: ["longhorn", "k8s", "kubernetes", "postgres"]
draft: false
---

# Building a K8s webapp using postgres running on Longhorn 

Setup up a Kubernetes (k8s) web application using PostgreSQL, with data stored on Longhorn. Longhorn is a lightweight, reliable, and powerful distributed block storage system for Kubernetes.

Here are the steps to achieve this setup:

## 1. Prerequisites
- A Kubernetes cluster up and running.
- kubectl configured to interact with your cluster.
- Helm installed (for easier application deployments).
- Longhorn installed and configured in your cluster.
- PostgreSQL Helm chart.
- Your web application container image.

![](/public/images/posts/postgres-on-longhorn.png)

## 2. Install an Ingress Controller
For this example, we will use NGINX Ingress Controller. You can install it using Helm:

```sh
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx
```

## 3. Install Longhorn
If you haven't installed Longhorn yet, you can do it using Helm:

```sh
helm repo add longhorn https://charts.longhorn.io
helm repo update
helm install longhorn longhorn/longhorn --namespace longhorn-system --create-namespace
```
After installing Longhorn, you can access the Longhorn UI to configure storage volumes.

## 4. Install PostgreSQL with Helm
Add the Bitnami repository and install PostgreSQL:

```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install postgres bitnami/postgresql
```
You can customize the PostgreSQL installation by providing a values file:

```sh
helm install postgres bitnami/postgresql -f values.yaml
```
An example values.yaml:

```yaml

primary:
  persistence:
    storageClass: "longhorn"
    size: 8Gi
```
This configuration sets Longhorn as the storage class and allocates an 8Gi volume for PostgreSQL.

You can get postgres DNS name and other info about the package:
```sh
helm status postgres
```

## 5. Deploy Your Web Application
Create a deployment, service and ingress for your web application. Here's an example of a simple web application deployment:

#### Step 1: Create the Kubernetes Secret

First, create a secret that contains your PostgreSQL credentials and database name. Save the following YAML to a file named postgres-secret.yaml:
**postgres-secret.yaml**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
  namespace: default
type: Opaque
data:
  database: cG9zdGdyZXM=    # 'postgres' base64 encoded
  username: cG9zdGdyZXM=    # 'postgres' base64 encoded
  password: cGFzc3dvcmQ=    # 'password' base64 encoded
```
Replace 'your-value' with your actual database name, username, or password.

Apply the secret:

```bash
kubectl apply -f postgres-secret.yaml
```
#### Step 2: Create the Kubernetes web app deployment manifest:

**web-app-deployment.yaml:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web-app
        image: your-web-app-image:latest
        ports:
        - containerPort: 8080
        env:
        - name: POSTGRES_DB
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: database
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: username
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        - name: POSTGRES_HOST
          value: postgres-postgresql.default.svc.cluster.local

```
You can get POSTGRES_HOST value from 
```sh
helm status postgres
```

#### Step 3: Create the Kubernetes web app service manifest:
**web-app-service.yaml:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app
spec:
  selector:
    app: web-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP

```
#### Step 4: Create the Kubernetes web app ingress manifest:
**web-app-ingress.yaml:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: your-domain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-app
            port:
              number: 80
```

#### Step 5: Apply the deployment, service and ingress:

```sh
kubectl apply -f web-app-deployment.yaml
kubectl apply -f web-app-service.yaml
kubectl apply -f web-app-ingress.yaml
```

## 6. Update Your DNS
Ensure your domain points to the external IP of your NGINX Ingress controller. You can find the IP using the following command:

```sh
kubectl get services -o wide -w --namespace default ingress-nginx-controller
```
Update your DNS settings to point your domain to this IP.
## 7. Access the Web Application
Once everything is deployed, you can access your web application through the LoadBalancer IP.

## 8. Verify Longhorn Volume Usage
You can check the Longhorn UI to verify that your PostgreSQL instance is using Longhorn volumes correctly.

## Summary
By following these steps, you should have a Kubernetes web application running with PostgreSQL using Longhorn for persistent storage. This setup ensures that your database has reliable and scalable storage while your application can scale independently.