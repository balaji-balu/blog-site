---
title: "Microservices: Service Discovery"
description: "meta description"
date: 2024-07-24T05:00:00Z
image: "/images/posts/01.jpg"
categories: ["microservices"]
authors: ["Balaji Balasundaram"]
tags: ["microservices", "service", "discovery", "kubernetes", "k8s",  "k3s", ]
draft: false
---


Service discovery is a crucial component in microservices architecture, enabling services to find and communicate with each other without hardcoding endpoints. Using Cilium and Envoy together can provide a robust solution for service discovery, leveraging Cilium’s advanced networking and security capabilities alongside Envoy’s powerful proxy features.

#### Overview
- Cilium: Provides eBPF-based networking, security, and observability for cloud-native environments. It simplifies network and security policies and offers deep visibility into network traffic.
- Envoy: A high-performance, open-source edge and service proxy designed for cloud-native applications. It handles service discovery, load balancing, and observability.
Service Discovery with Cilium and Envoy
Setup Cilium: Ensure that Cilium is deployed in your Kubernetes cluster. Cilium provides the network connectivity and security for your microservices.

Install Envoy: Deploy Envoy as a sidecar proxy to your microservices or as a gateway for your services. Envoy handles the service discovery and routing.

Configure Service Discovery:

Cilium Service Load Balancer: Cilium can be configured to act as a service load balancer, enabling services to discover each other within the cluster.
Envoy Service Discovery: Envoy can be configured to use different service discovery mechanisms, such as DNS, EDS (Envoy Discovery Service), or static configuration.
Steps to Implement Service Discovery
Step 1: Deploy Cilium
Deploy Cilium in your Kubernetes cluster by following the official Cilium installation guide. Ensure Cilium is properly configured to manage networking and security policies.

bash
Copy code
kubectl create namespace cilium
helm install cilium cilium/cilium --namespace cilium
Step 2: Install Envoy
Deploy Envoy either as a sidecar or as a standalone gateway. For a sidecar setup, inject Envoy into your microservices pods.

Example deployment configuration:

yaml
Copy code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-service
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: my-service
    spec:
      containers:
      - name: my-service
        image: my-service:latest
      - name: envoy
        image: envoyproxy/envoy:v1.20.0
        ports:
        - containerPort: 9901
        volumeMounts:
        - name: envoy-config
          mountPath: /etc/envoy
      volumes:
      - name: envoy-config
        configMap:
          name: envoy-config
Step 3: Configure Envoy for Service Discovery
Create an Envoy configuration that specifies how services are discovered. Use Cilium's capabilities to manage service discovery endpoints.

Example Envoy configuration:

yaml
Copy code
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address:
        address: 0.0.0.0
        port_value: 10000
    filter_chains:
    - filters:
      - name: envoy.filters.network.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
          stat_prefix: ingress_http
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match:
                  prefix: "/"
                route:
                  cluster: service_cluster
          http_filters:
          - name: envoy.filters.http.router
  clusters:
  - name: service_cluster
    connect_timeout: 0.25s
    type: STRICT_DNS
    dns_lookup_family: V4_ONLY
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: service_cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: my-service.default.svc.cluster.local
                port_value: 80
Step 4: Integrate Cilium with Envoy
Ensure Cilium is managing network policies and service load balancing. You can use Cilium’s eBPF-based service load balancer to manage service endpoints dynamically.

Example Cilium Network Policy:

yaml
Copy code
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: my-service-policy
spec:
  endpointSelector:
    matchLabels:  
      app: my-service
  ingress:
  - fromEndpoints:
    - matchLabels:
        app: my-client
    toPorts:
    - ports:
      - port: "80"
        protocol: TCP
Benefits
Scalability: Automatically discovers and scales services without manual endpoint configuration.
Security: Cilium’s eBPF-based policies provide fine-grained security controls.
Observability: Combined with Envoy’s observability features, this setup offers deep insights into service communication and performance.
By leveraging Cilium and Envoy for service discovery, you can build a scalable, secure, and observable microservices architecture.

