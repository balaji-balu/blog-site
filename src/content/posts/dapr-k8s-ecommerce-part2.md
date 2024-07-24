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
# Ecommerce app in go(part 2): k8s, Dapr, Redis

Let's expand the e-commerce microservices application by adding Payment, Inventory, and Shipping services, and integrate them using Dapr's capabilities. We'll use service invocation for direct communication between services and pub/sub for event-driven communication.

Payment Service
The Payment Service will handle payment processing.

go
Copy code
// payment-service/main.go
package main

import (
    "encoding/json"
    "context"
    "github.com/dapr/go-sdk/service/common"
    "github.com/dapr/go-sdk/service/http"
    "net/http"
)

type Payment struct {
    OrderID   string `json:"order_id"`
    UserID    string `json:"user_id"`
    Amount    float64 `json:"amount"`
    Status    string `json:"status"`
}

var payments = map[string]Payment{}

func main() {
    svc := http.NewService(":8083")

    svc.AddServiceInvocationHandler("/processPayment", processPayment)

    if err := svc.Start(); err != nil {
        panic(err)
    }
}

func processPayment(ctx context.Context, in *common.InvocationEvent) (*common.Content, error) {
    var payment Payment
    if err := json.Unmarshal(in.Data, &payment); err != nil {
        return nil, err
    }
    payment.Status = "processed"
    payments[payment.OrderID] = payment

    // Publish an event after payment is processed
    daprClient, err := client.NewClient()
    if err != nil {
        return nil, err
    }
    defer daprClient.Close()
    paymentEvent, _ := json.Marshal(payment)
    err = daprClient.PublishEvent(ctx, "order-pubsub", "payment-processed", paymentEvent)
    if err != nil {
        return nil, err
    }

    return &common.Content{
        Data:        in.Data,
        ContentType: "application/json",
    }, nil
}
Inventory Service
The Inventory Service will manage product inventory.

go
Copy code
// inventory-service/main.go
package main

import (
    "encoding/json"
    "context"
    "github.com/dapr/go-sdk/service/common"
    "github.com/dapr/go-sdk/service/http"
    "github.com/dapr/go-sdk/client"
    "net/http"
)

type Inventory struct {
    ProductID string `json:"product_id"`
    Quantity  int    `json:"quantity"`
}

var inventories = map[string]Inventory{}

func main() {
    svc := http.NewService(":8084")

    svc.AddServiceInvocationHandler("/updateInventory", updateInventory)
    svc.AddTopicEventHandler("order-pubsub", "order-created", orderCreatedHandler)

    if err := svc.Start(); err != nil {
        panic(err)
    }
}

func updateInventory(ctx context.Context, in *common.InvocationEvent) (*common.Content, error) {
    var inventory Inventory
    if err := json.Unmarshal(in.Data, &inventory); err != nil {
        return nil, err
    }
    inventories[inventory.ProductID] = inventory
    return &common.Content{
        Data:        in.Data,
        ContentType: "application/json",
    }, nil
}

func orderCreatedHandler(ctx context.Context, e *common.TopicEvent) (retry bool, err error) {
    var order Order
    if err := json.Unmarshal(e.Data, &order); err != nil {
        return false, err
    }

    // Update inventory
    inventory, exists := inventories[order.ProductID]
    if exists {
        inventory.Quantity -= order.Quantity
        inventories[order.ProductID] = inventory
    }

    return false, nil
}
Shipping Service
The Shipping Service will handle shipping logistics.

go
Copy code
// shipping-service/main.go
package main

import (
    "encoding/json"
    "context"
    "github.com/dapr/go-sdk/service/common"
    "github.com/dapr/go-sdk/service/http"
    "github.com/dapr/go-sdk/client"
    "net/http"
)

type Shipping struct {
    OrderID   string `json:"order_id"`
    UserID    string `json:"user_id"`
    Address   string `json:"address"`
    Status    string `json:"status"`
}

var shipments = map[string]Shipping{}

func main() {
    svc := http.NewService(":8085")

    svc.AddServiceInvocationHandler("/createShipment", createShipment)
    svc.AddTopicEventHandler("order-pubsub", "payment-processed", paymentProcessedHandler)

    if err := svc.Start(); err != nil {
        panic(err)
    }
}

func createShipment(ctx context.Context, in *common.InvocationEvent) (*common.Content, error) {
    var shipping Shipping
    if err := json.Unmarshal(in.Data, &shipping); err != nil {
        return nil, err
    }
    shipping.Status = "shipped"
    shipments[shipping.OrderID] = shipping
    return &common.Content{
        Data:        in.Data,
        ContentType: "application/json",
    }, nil
}

func paymentProcessedHandler(ctx context.Context, e *common.TopicEvent) (retry bool, err error) {
    var payment Payment
    if err := json.Unmarshal(e.Data, &payment); err != nil {
        return false, err
    }

    // Create a shipment after payment is processed
    shipment := Shipping{
        OrderID: payment.OrderID,
        UserID:  payment.UserID,
        Address: "User Address", // Assuming address is fetched from user service
        Status:  "ready to ship",
    }
    shipments[shipment.OrderID] = shipment

    return false, nil
}
Pub/Sub Configuration
Create a pub/sub component configuration for Dapr. Save this as components/pubsub.yaml.

yaml
Copy code
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: order-pubsub
  namespace: default
spec:
  type: pubsub.redis
  version: v1
  metadata:
  - name: redisHost
    value: "localhost:6379"
  - name: redisPassword
    value: ""
Running Services
Create Dapr configurations and run each service.

Payment Service Configuration
json
Copy code
// config/payment-service-config.json
{
  "app-id": "payment-service",
  "app-port": "8083",
  "dapr-http-port": "3503"
}
Inventory Service Configuration
json
Copy code
// config/inventory-service-config.json
{
  "app-id": "inventory-service",
  "app-port": "8084",
  "dapr-http-port": "3504"
}
Shipping Service Configuration
json
Copy code
// config/shipping-service-config.json
{
  "app-id": "shipping-service",
  "app-port": "8085",
  "dapr-http-port": "3505"
}
Start the services:

sh
Copy code
# Start Payment Service
dapr run --config ./config/payment-service-config.json -- go run payment-service/main.go

# Start Inventory Service
dapr run --config ./config/inventory-service-config.json -- go run inventory-service/main.go

# Start Shipping Service
dapr run --config ./config/shipping-service-config.json -- go run shipping-service/main.go
Updating Order Service to Publish Events
Update the Order Service to publish events when an order is created.

go
Copy code
// order-service/main.go (updated)
package main

import (
    "encoding/json"
    "context"
    "github.com/dapr/go-sdk/service/common"
    "github.com/dapr/go-sdk/service/http"
    "github.com/dapr/go-sdk/client"
    "net/http"
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

    // Publish an event after order is created
    daprClient, err := client.NewClient()
    if err != nil {
        return nil, err
    }
    defer daprClient.Close()
    orderEvent, _ := json.Marshal(order)
    err = daprClient.PublishEvent(ctx, "order-pubsub", "order-created", orderEvent)
    if err != nil {
        return nil, err
    }

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
Summary
This setup now includes:

Payment Service: Handles payment processing and publishes events.
Inventory Service: Manages inventory and listens for order creation events.
Shipping Service: Manages shipping and listens for payment processed events.
All services communicate through Dapr's service invocation and pub/sub capabilities. This setup demonstrates the power and simplicity of using Dapr to build a robust, event-driven microservices architecture.