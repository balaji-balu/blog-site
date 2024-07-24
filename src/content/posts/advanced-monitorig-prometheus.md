---
title: Advanced Monitoring with Prometheus and Grafana
description: "meta description"
date: 2024-07-08T06:00:00+00:00
image: "/images/posts/03.jpg"
categories: ["observability"]
authors: ["Balaji Balasundaram"]
tags: ["k3s", "observability"]
draft: false
---

# Advanced Monitoring with Prometheus and Grafana
For more advanced monitoring, integrating Prometheus and Grafana provides a powerful combination for metrics collection, alerting, and visualization.

### Step 1: Deploy Prometheus
Prometheus collects and stores metrics data. Create a file named prometheus-deployment.yaml with the following content:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    scrape_configs:
      - job_name: 'kubernetes-apiservers'
        kubernetes_sd_configs:
          - role: endpoints
        relabel_configs:
          - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
            action: keep
            regex: default;kubernetes;https

---

apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus
  namespace: monitoring

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      serviceAccountName: prometheus
      containers:
      - name: prometheus
        image: prom/prometheus:latest
        args:
          - --config.file=/etc/prometheus/prometheus.yml
        ports:
        - containerPort: 9090
        volumeMounts:
        - name: prometheus-config-volume
          mountPath: /etc/prometheus
      volumes:
      - name: prometheus-config-volume
        configMap:
          name: prometheus-config

---

apiVersion: v1
kind: Service
metadata:
  name: prometheus
  namespace: monitoring
spec:
  type: NodePort
  ports:
  - port: 9090
    targetPort: 9090
    nodePort: 30090
  selector:
    app: prometheus
```
Apply the Prometheus deployment:

```sh
sudo k3s kubectl apply -f prometheus-deployment.yaml
```
### Step 2: Deploy Grafana
Grafana visualizes the metrics collected by Prometheus. Create a file named grafana-deployment.yaml with the following content:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-datasources
  namespace: monitoring
data:
  prometheus-datasource.yaml: |
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      access: proxy
      url: http://prometheus:9090

---

apiVersion: v1
kind: ServiceAccount
metadata:
  name: grafana
  namespace: monitoring

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      serviceAccountName: grafana
      containers:
      - name: grafana
        image: grafana/grafana:latest
        ports:
        - containerPort: 3000
        volumeMounts:
        - name: grafana-storage
          mountPath: /var/lib/grafana
        - name: grafana-datasources
          mountPath: /etc/grafana/provisioning/datasources
          subPath: prometheus-datasource.yaml
      volumes:
      - name: grafana-storage
        emptyDir: {}
      - name: grafana-datasources
        configMap:
          name: grafana-datasources

---

apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: monitoring
spec:
  type: NodePort
  ports:
  - port: 3000
    targetPort: 3000
    nodePort: 30030
  selector:
    app: grafana
```
Apply the Grafana deployment:

```sh
sudo k3s kubectl apply -f grafana-deployment.yaml
```
### Accessing Prometheus and Grafana
Prometheus will be available at http://<NODE_IP>:30090, and Grafana at http://<NODE_IP>:30030.

### Setting Up Grafana Dashboards
Open Grafana in your browser and log in (default username: admin, password: admin).
Add the Prometheus data source (configured by the ConfigMap).
Import pre-built Kubernetes dashboards from the Grafana Dashboard repository.