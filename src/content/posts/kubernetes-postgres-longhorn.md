---
title: "Kubernetes: Connect Postgresql using Longhorn"
description: "meta description"
date: 2024-07-19T05:00:00Z
image: "/images/posts/01.jpg"
categories: ["kubernetes", "os"]
authors: ["Balaji Balasundaram"]
tags: ["kubernetes", "k8s", "k3s", "postres", "longhorn", "postgresql", "storage"]
draft: false
---

## Kubernetes: Connect Postgresql with Longhorn

To connect PostgreSQL with Longhorn for persistent storage in a Kubernetes or k3s cluster, follow these steps:

### 1. Install Longhorn
If you haven’t already installed Longhorn, you can do so using Helm or by applying YAML manifests. Longhorn is a cloud-native distributed block storage system designed for Kubernetes.

#### Using Helm:

Add the Longhorn Helm repository:

```bash
helm repo add longhorn https://charts.longhorn.io
```
Update your Helm repositories:

```bash
helm repo update
```
Install Longhorn:

```bash
helm install longhorn longhorn/longhorn --namespace longhorn-system --create-namespace
```
#### Using YAML manifests:

Apply the Longhorn installation manifest:
```bash
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.3.1/deploy/longhorn.yaml
```

### 2. Create a StorageClass for Longhorn
Longhorn will typically create a default StorageClass upon installation, but you can create or customize one if needed.

Here’s an example StorageClass for Longhorn:

**longhorn-storage.yaml**

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: longhorn
provisioner: driver.longhorn.io
```
### 3. Create a PersistentVolumeClaim (PVC) Using Longhorn Storage
Define a PVC to request storage from Longhorn:

**postgres-pvc.yaml**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: longhorn
```
### 4. Deploy PostgreSQL
Deploy PostgreSQL using the PVC you created. Here’s an example of a PostgreSQL deployment that uses Longhorn for persistent storage:

#### Deployment:

**postgres-deployment.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:14
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_DB
          value: mydatabase
        - name: POSTGRES_USER
          value: myuser
        - name: POSTGRES_PASSWORD
          value: mypassword
        volumeMounts:
        - mountPath: /var/lib/postgresql/data
          name: postgres-storage
      volumes:
      - name: postgres-storage
        persistentVolumeClaim:
          claimName: postgres-pvc
```
#### Service:

Expose PostgreSQL with a Service:

**postgres-service**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
spec:
  selector:
    app: postgres
  ports:
  - protocol: TCP
    port: 5432
    targetPort: 5432
```

### 5. Install
```sh
kubectl apply -f longhorn-storage.yaml
kubectl apply -f postgres-pvc.yaml
kubectl apply -f postgres-service.yaml
kubectl apply -f postgres-deployment.yaml
```

Kubernetes will create `pv` using `postgres-pvc` 

### 6. Verify and Access
Check the status of your deployment and services:

```bash
kubectl get pods
kubectl get svc
kubectl logs <postgres-pod-name>
```
You can connect to PostgreSQL from within your cluster using:

```bash
psql -h postgres-service -U myuser -d mydatabase
```
### 7. Test with an application

Deploy your Go application in the **same Kubernetes cluster**. Your Go application will use the Kubernetes Service name to connect to PostgreSQL.

#### Deployment Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-app-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: go-app
  template:
    metadata:
      labels:
        app: go-app
    spec:
      containers:
      - name: go-app
        image: your-go-app-image
        env:
        - name: POSTGRES_HOST
          value: "postgres-service"
        - name: POSTGRES_PORT
          value: "5432"
        - name: POSTGRES_USER
          value: "myuser"
        - name: POSTGRES_PASSWORD
          value: "mypassword"
        - name: POSTGRES_DB
          value: "mydatabase"
```

#### Go Application Code:

Update your Go application to read PostgreSQL connection details from environment variables.

```go
package main

import (
    "database/sql"
    "fmt"
    "log"
    "os"

    _ "github.com/lib/pq"
)

func main() {
    host := os.Getenv("POSTGRES_HOST")
    port := os.Getenv("POSTGRES_PORT")
    user := os.Getenv("POSTGRES_USER")
    password := os.Getenv("POSTGRES_PASSWORD")
    dbname := os.Getenv("POSTGRES_DB")

    psqlInfo := fmt.Sprintf("host=%s port=%s user=%s password=%s dbname=%s sslmode=disable",
        host, port, user, password, dbname)

    db, err := sql.Open("postgres", psqlInfo)
    if err != nil {
        log.Fatalf("Error opening database: %q", err)
    }
    defer db.Close()

    err = db.Ping()
    if err != nil {
        log.Fatalf("Error connecting to the database: %q", err)
    }

    fmt.Println("Successfully connected!")

    var version string
    err = db.QueryRow("SELECT version()").Scan(&version)
    if err != nil {
        log.Fatalf("Error executing query: %q", err)
    }
    fmt.Printf("PostgreSQL version: %s\n", version)
}
```

```sh
go mod init testapp
go mod tidy
go run main.go
```
Output:
```text
Successfully connected!
PostgreSQL version: 14
```

#### Notes:
- Longhorn UI: You can access the Longhorn UI to manage and monitor volumes. Typically, it’s exposed through a LoadBalancer or NodePort service. You might need to configure an access method based on your cluster setup.
- Persistence: Ensure that Longhorn volumes are correctly provisioned and attached to your PostgreSQL pod to maintain data integrity and availability.

This setup should provide a robust environment for running PostgreSQL with Longhorn as the storage backend in your Kubernetes or k3s cluster.


