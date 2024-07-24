---
title: "Logistics microservices: DevSpace"
description: "meta description"
date: 2024-07-21T05:00:00Z
image: "/images/posts/01.jpg"
categories: ["kubernetes", "os"]
authors: ["Balaji Balasundaram"]
tags: ["microservices", "logistics", "kubernetes", "k8s", "devspace"]
draft: false
---

Integrating DevSpace into the deployment workflow will streamline the development and deployment processes, allowing for efficient local development and automated deployment to Kubernetes clusters. Below is a revised architecture and workflow using DevSpace.

1. Project Structure
```plaintext
Copy code
logistics-project/
├── api-gateway/
│   ├── Dockerfile
│   ├── devspace.yaml
│   ├── gateway/
│   │   ├── main.go
│   │   ├── config/
│   │   │   └── config.go
│   │   ├── routes/
│   │   │   └── routes.go
│   │   ├── middleware/
│   │   │   └── auth.go
├── services/
│   ├── order-service/
│   │   ├── Dockerfile
│   │   ├── devspace.yaml
│   │   ├── order/
│   │   │   ├── main.go
│   │   │   ├── controllers/
│   │   │   │   └── order_controller.go
│   │   │   ├── models/
│   │   │   │   └── order.go
│   │   │   ├── repositories/
│   │   │   │   └── order_repository.go
│   │   │   ├── services/
│   │   │   │   └── order_service.go
│   ├── inventory-service/
│   │   ├── Dockerfile
│   │   ├── devspace.yaml
│   │   ├── inventory/
│   │   │   ├── main.go
│   │   │   ├── controllers/
│   │   │   │   └── inventory_controller.go
│   │   │   ├── models/
│   │   │   │   └── inventory.go
│   │   │   ├── repositories/
│   │   │   │   └── inventory_repository.go
│   │   │   ├── services/
│   │   │   │   └── inventory_service.go
│   ├── shipping-service/
│   │   ├── Dockerfile
│   │   ├── devspace.yaml
│   │   ├── shipping/
│   │   │   ├── main.go
│   │   │   ├── controllers/
│   │   │   │   └── shipping_controller.go
│   │   │   ├── models/
│   │   │   │   └── shipping.go
│   │   │   ├── repositories/
│   │   │   │   └── shipping_repository.go
│   │   │   ├── services/
│   │   │   │   └── shipping_service.go
├── config/
│   ├── Dockerfile
│   ├── devspace.yaml
│   ├── config-service/
│   │   ├── main.go
│   │   ├── config/
│   │   │   └── config.go
├── auth/
│   ├── Dockerfile
│   ├── devspace.yaml
│   ├── auth-service/
│   │   ├── main.go
│   │   ├── controllers/
│   │   │   └── auth_controller.go
│   │   ├── models/
│   │   │   └── user.go
│   │   ├── repositories/
│   │   │   └── user_repository.go
│   │   ├── services/
│   │   │   └── auth_service.go
├── common/
│   ├── utils/
│   │   └── utils.go
│   ├── middleware/
│   │   └── common_middleware.go
├── deployments/
│   ├── api-gateway/
│   │   └── deployment.yaml
│   │   └── service.yaml
│   ├── order-service/
│   │   └── deployment.yaml
│   │   └── service.yaml
│   ├── inventory-service/
│   │   └── deployment.yaml
│   │   └── service.yaml
│   ├── shipping-service/
│   │   └── deployment.yaml
│   │   └── service.yaml
│   ├── auth-service/
│   │   └── deployment.yaml
│   │   └── service.yaml
│   ├── config-service/
│   │   └── deployment.yaml
│   │   └── service.yaml
├── scripts/
│   ├── build.sh
│   ├── deploy.sh
├── docs/
│   ├── api/
│   │   └── api_docs.md
├── tests/
│   ├── integration/
│   │   └── integration_tests.go
│   ├── unit/
│   │   └── unit_tests.go
├── .gitignore
├── docker-compose.yml
├── README.md
```
2. DevSpace Configuration for API Gateway (api-gateway/devspace.yaml)
yaml
Copy code
version: v1beta10
name: api-gateway
deployments:
  - name: api-gateway
    kubectl:
      manifests:
        - deployment.yaml
        - service.yaml
images:
  api-gateway:
    image: api-gateway:latest
    dockerfile: Dockerfile
    context: .
dev:
  autoReload:
    deployments:
      - name: api-gateway
3. DevSpace Configuration for Order Service (services/order-service/devspace.yaml)
yaml
Copy code
version: v1beta10
name: order-service
deployments:
  - name: order-service
    kubectl:
      manifests:
        - deployment.yaml
        - service.yaml
images:
  order-service:
    image: order-service:latest
    dockerfile: Dockerfile
    context: .
dev:
  autoReload:
    deployments:
      - name: order-service
4. DevSpace Configuration for Inventory Service (services/inventory-service/devspace.yaml)
yaml
Copy code
version: v1beta10
name: inventory-service
deployments:
  - name: inventory-service
    kubectl:
      manifests:
        - deployment.yaml
        - service.yaml
images:
  inventory-service:
    image: inventory-service:latest
    dockerfile: Dockerfile
    context: .
dev:
  autoReload:
    deployments:
      - name: inventory-service
5. DevSpace Configuration for Shipping Service (services/shipping-service/devspace.yaml)
yaml
Copy code
version: v1beta10
name: shipping-service
deployments:
  - name: shipping-service
    kubectl:
      manifests:
        - deployment.yaml
        - service.yaml
images:
  shipping-service:
    image: shipping-service:latest
    dockerfile: Dockerfile
    context: .
dev:
  autoReload:
    deployments:
      - name: shipping-service
6. DevSpace Configuration for Auth Service (auth/devspace.yaml)
yaml
Copy code
version: v1beta10
name: auth-service
deployments:
  - name: auth-service
    kubectl:
      manifests:
        - deployment.yaml
        - service.yaml
images:
  auth-service:
    image: auth-service:latest
    dockerfile: Dockerfile
    context: .
dev:
  autoReload:
    deployments:
      - name: auth-service
7. DevSpace Configuration for Config Service (config/devspace.yaml)
yaml
Copy code
version: v1beta10
name: config-service
deployments:
  - name: config-service
    kubectl:
      manifests:
        - deployment.yaml
        - service.yaml
images:
  config-service:
    image: config-service:latest
    dockerfile: Dockerfile
    context: .
dev:
  autoReload:
    deployments:
      - name: config-service

### 8. Kubernetes Deployments and Services

Deployments and services for each microservice should already be defined in the deployments/ directory as outlined previously. Here are examples for reference:

##### Deployment for API Gateway (deployments/api-gateway/deployment.yaml)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway-deployment
  labels:
    app: api-gateway
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api-gateway
  template:
    metadata:
      labels:
        app: api-gateway
    spec:
      containers:
      - name: api-gateway
        image: api-gateway:latest
        ports:
        - containerPort: 8080
        env:
        - name: CONFIG_SERVICE_URL
          value: "http://config-service:8082"
```
##### Service for API Gateway (deployments/api-gateway/service.yaml)
yaml
Copy code
apiVersion: v1
kind: Service
metadata:
  name: api-gateway-service
spec:
  selector:
    app: api-gateway
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer
Similar deployment and service configurations should be created for each microservice.

### 9. Build Script (scripts/build.sh)
```bash
#!/bin/bash

# Build API Gateway
cd api-gateway
devspace build --image api-gateway

# Build Order Service
cd ../services/order-service
devspace build --image order-service

# Build Inventory Service
cd ../inventory-service
devspace build --image inventory-service

# Build Shipping Service
cd ../shipping-service
devspace build --image shipping-service

# Build Auth Service
cd ../../auth
devspace build --image auth-service

# Build Config Service
cd ../config
devspace build --image config-service
```
10. Deployment Script (scripts/deploy.sh)

```bash
Copy code
#!/bin/bash

# Deploy API Gateway
cd api-gateway
devspace deploy

# Deploy Order Service
cd ../services/order-service
devspace deploy

# Deploy Inventory Service
cd ../inventory-service
devspace deploy

# Deploy Shipping Service
cd ../shipping-service
devspace deploy

# Deploy Auth Service
cd ../../auth
devspace deploy

# Deploy Config Service
cd ../config
devspace deploy
```
### 11. Local Development with DevSpace
To start developing locally with DevSpace, follow these steps:

Install DevSpace if you haven't already:

```sh
curl -s -L "https://github.com/loft-sh/devspace/releases/latest/download/devspace-linux-amd64" -o /usr/local/bin/devspace
chmod +x /usr/local/bin/devspace
```
Start DevSpace in development mode for each service. For example, to start the API Gateway:

```sh
cd api-gateway
devspace dev
```
This will build the Docker image, deploy the service to your Kubernetes cluster, and set up port-forwarding and file synchronization for rapid local development.

By incorporating DevSpace into your deployment workflow, you can achieve a more streamlined and automated development process, allowing developers to focus on writing code while handling the complexities of Kubernetes deployment and management in the background.

