---
title: "Testing k8s HPA"
description: "meta description"
date: 2024-07-24T05:00:00Z
image: "/images/posts/01.jpg"
categories: ["kubernetes"]
authors: ["Balaji Balasundaram"]
tags: ["kubernetes", "k8s", "testing", "hpa"]
draft: false
---

Testing Kubernetes Horizontal Pod Autoscaler (HPA) involves simulating a load to see if the HPA scales your application pods as expected. Here’s a step-by-step guide on how to do it:

#### Set Up Your Kubernetes Cluster:
Ensure you have a running Kubernetes cluster. You can use Minikube, kind, or a managed Kubernetes service like GKE, EKS, or AKS.

#### Deploy an Application:
Deploy a sample application to your cluster. For this example, let’s use an Nginx deployment.

```yaml
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
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
Apply this deployment:

```sh
kubectl apply -f nginx-deployment.yaml
```
Expose the Deployment:
Create a service to expose the Nginx deployment.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```
Apply this service:

```sh
kubectl apply -f nginx-service.yaml
```
#### Create an HPA Resource:
Create an HPA resource to scale the Nginx deployment based on CPU usage.

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```
Apply this HPA:

```sh
kubectl apply -f nginx-hpa.yaml
```
#### Generate Load:
Use a tool like Apache Bench (ab) or hey to generate load on your application.

```sh
hey -z 5m -c 50 http://<EXTERNAL-IP>
```
Replace <EXTERNAL-IP> with the external IP of the nginx-service.

#### Monitor HPA:
Monitor the HPA to see if it scales the pods as expected.

```sh
kubectl get hpa -w
```
This command will watch the HPA resource and show you the scaling activity in real-time.

#### Verify the Scaling:
Check the number of pods running to verify that scaling has occurred.

```sh
kubectl get pods -l app=nginx
```

#### Optional: Automating with Scripts
You can automate this process with scripts to streamline your testing. Here’s an example of a Bash script to set up the environment and test the HPA:

```sh
#!/bin/bash

# Deploy Nginx
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml
kubectl apply -f nginx-hpa.yaml

# Wait for the deployment to be available
kubectl rollout status deployment/nginx-deployment

# Generate load
EXTERNAL_IP=$(kubectl get svc/nginx-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
hey -z 5m -c 50 http://$EXTERNAL_IP

# Monitor HPA
kubectl get hpa -w
```
This script deploys the resources, waits for the deployment to be ready, generates load, and then monitors the HPA.

By following these steps, you can effectively test the scaling behavior of your Kubernetes HPA.