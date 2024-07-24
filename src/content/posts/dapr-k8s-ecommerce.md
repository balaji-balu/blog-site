---
title: ""
description: "meta description"
date: 2024-07-14T05:00:00Z
image: "/images/posts/01.jpg"
categories: ["kubernetes"]
authors: ["Balaji Balasundaram"]
tags: ["kubernetes", "k8s", "dapr", "cluster"]
draft: true
---
# Ecommerce app in go: k8s, Dapr, Redis

Creating a complex e-commerce microservices application using Dapr (Distributed Application Runtime) involves several components and services to handle different functionalities such as user management, product catalog, order processing, payment processing, and more. Dapr provides building blocks for microservices such as service invocation, state management, pub/sub messaging, and bindings, which simplify the development of distributed applications.

Here's a high-level outline of the microservices you might need and a sample implementation for some of them using Dapr:

## High-Level Architecture
- User Service: Manages user accounts and authentication.
- Product Service: Manages product information.
- Order Service: Handles order creation, processing, and tracking.
- Payment Service: Manages payment processing.
- Inventory Service: Manages product inventory.
- Shipping Service: Handles shipping logistics.
- Notification Service: Sends notifications to users.
Each service will communicate with others through Dapr's service invocation, state management, and pub/sub features.

Step-by-Step Implementation
1. Set Up Dapr
First, install Dapr and initialize it in your development environment.

sh
Copy code
# Install Dapr CLI
wget -q https://raw.githubusercontent.com/dapr/cli/master/install/install.sh -O - | /bin/bash

# Initialize Dapr
dapr init
2. Define Microservices
We'll start by defining a few essential microservices: User Service, Product Service, and Order Service.

User Service
A simple user service that manages user accounts.

go
Copy code
// user-service/main.go
package main

import (
    "encoding/json"
    "net/http"
    "github.com/gorilla/mux"
    "github.com/dapr/go-sdk/service/common"
    "github.com/dapr/go-sdk/service/http"
)

type User struct {
    ID    string `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

var users = map[string]User{}

func main() {
    svc := http.NewService(":8080")

    svc.AddServiceInvocationHandler("/createUser", createUser)
    svc.AddServiceInvocationHandler("/getUser", getUser)

    if err := svc.Start(); err != nil {
        panic(err)
    }
}

func createUser(ctx context.Context, in *common.InvocationEvent) (*common.Content, error) {
    var user User
    if err := json.Unmarshal(in.Data, &user); err != nil {
        return nil, err
    }
    users[user.ID] = user
    return &common.Content{
        Data:        in.Data,
        ContentType: "application/json",
    }, nil
}

func getUser(ctx context.Context, in *common.InvocationEvent) (*common.Content, error) {
    id := string(in.Data)
    user, exists := users[id]
    if !exists {
        return &common.Content{
            Data:        nil,
            ContentType: "application/json",
        }, nil
    }
    userData, _ := json.Marshal(user)
    return &common.Content{
        Data:        userData,
        ContentType: "application/json",
    }, nil
}
Product Service
A service to manage products.

go
Copy code
// product-service/main.go
package main

import (
    "encoding/json"
    "net/http"
    "github.com/gorilla/mux"
    "github.com/dapr/go-sdk/service/common"
    "github.com/dapr/go-sdk/service/http"
)

type Product struct {
    ID    string  `json:"id"`
    Name  string  `json:"name"`
    Price float64 `json:"price"`
}

var products = map[string]Product{}

func main() {
    svc := http.NewService(":8081")

    svc.AddServiceInvocationHandler("/createProduct", createProduct)
    svc.AddServiceInvocationHandler("/getProduct", getProduct)

    if err := svc.Start(); err != nil {
        panic(err)
    }
}

func createProduct(ctx context.Context, in *common.InvocationEvent) (*common.Content, error) {
    var product Product
    if err := json.Unmarshal(in.Data, &product); err != nil {
        return nil, err
    }
    products[product.ID] = product
    return &common.Content{
        Data:        in.Data,
        ContentType: "application/json",
    }, nil
}

func getProduct(ctx context.Context, in *common.InvocationEvent) (*common.Content, error) {
    id := string(in.Data)
    product, exists := products[id]
    if !exists {
        return &common.Content{
            Data:        nil,
            ContentType: "application/json",
        }, nil
    }
    productData, _ := json.Marshal(product)
    return &common.Content{
        Data:        productData,
        ContentType: "application/json",
    }, nil
}
Order Service
A service to manage orders.

go
Copy code
// order-service/main.go
package main

import (
    "encoding/json"
    "net/http"
    "github.com/gorilla/mux"
    "github.com/dapr/go-sdk/service/common"
    "github.com/dapr/go-sdk/service/http"
)

type Order struct {
    ID       string `json:"id"`
    UserID   string `json:"user_id"`
    ProductID string `json:"product_id"`
    Quantity int    `json:"quantity"`
    Status   string `json:"status"`
}

var orders = map[string]Order{}

func main() {
    svc := http.NewService(":8082")

    svc.AddServiceInvocationHandler("/createOrder", createOrder)
    svc.AddServiceInvocationHandler("/getOrder", getOrder)

    if err := svc.Start(); err != nil {
        panic(err)
    }
}

func createOrder(ctx context.Context, in *common.InvocationEvent) (*common.Content, error) {
    var order Order
    if err := json.Unmarshal(in.Data, &order); err != nil {
        return nil, err
    }
    order.Status = "created"
    orders[order.ID] = order
    return &common.Content{
        Data:        in.Data,
        ContentType: "application/json",
    }, nil
}

func getOrder(ctx context.Context, in *common.InvocationEvent) (*common.Content, error) {
    id := string(in.Data)
    order, exists := orders[id]
    if !exists {
        return &common.Content{
            Data:        nil,
            ContentType: "application/json",
        }, nil
    }
    orderData, _ := json.Marshal(order)
    return &common.Content{
        Data:        orderData,
        ContentType: "application/json",
    }, nil
}
3. Run Services with Dapr
Create dapr configurations and run each service with Dapr.

User Service Configuration
json
Copy code
// config/user-service-config.json
{
  "app-id": "user-service",
  "app-port": "8080",
  "dapr-http-port": "3500"
}
Product Service Configuration
json
Copy code
// config/product-service-config.json
{
  "app-id": "product-service",
  "app-port": "8081",
  "dapr-http-port": "3501"
}
Order Service Configuration
json
Copy code
// config/order-service-config.json
{
  "app-id": "order-service",
  "app-port": "8082",
  "dapr-http-port": "3502"
}
4. Start Services
sh
Copy code
# Start User Service
dapr run --config ./config/user-service-config.json -- go run user-service/main.go

# Start Product Service
dapr run --config ./config/product-service-config.json -- go run product-service/main.go

# Start Order Service
dapr run --config ./config/order-service-config.json -- go run order-service/main.go
5. Invoke Services
You can invoke these services using Dapr's HTTP API. For example, to create a user:

sh
Copy code
curl -X POST http://localhost:3500/v1.0/invoke/user-service/method/createUser -d '{
  "id": "1",
  "name": "John Doe",
  "email": "john.doe@example.com"
}'
Similarly, you can invoke other methods for different services.

6. Add More Services and Integrate
Follow the same pattern to add more services (e.g., Payment, Inventory, Shipping) and integrate them using Dapr's service invocation, pub/sub, and state management capabilities.

This is a simplified example to get you started. For a full-fledged application, you'll need to implement more robust error handling, security, and other production-grade features.