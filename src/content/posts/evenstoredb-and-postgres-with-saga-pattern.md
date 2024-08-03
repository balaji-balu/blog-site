---
title: "EventstoreDB and Postgres with Saga pattern"
description: "meta description"
date: 2024-08-03T05:00:00Z
image: "/images/posts/Saga-pattern.png"
categories: ["kubernetes", "os"]
authors: ["Balaji Balasundaram"]
tags: ["event-driven", "architecture", "eventstoredb", "microservices", "postgres", "saga-pattern"]
draft: false
---
courtesy: [Couchbase](https://www.couchbase.com/blog/saga-pattern-implement-business-transactions-using-microservices-part-2/)

Let's dive into an example of implementing the Saga pattern using EventStoreDB and PostgreSQL in a microservices architecture.

#### Saga Pattern Overview
The Saga pattern is a way to manage complex, long-running transactions in a microservices architecture. Instead of a single transaction spanning multiple services, a saga is a series of local transactions. Each local transaction updates a service and publishes an event or message. The saga orchestrator (or coordinator) listens for these events and decides what the next step is. If something goes wrong, the saga ensures that compensating transactions are executed to undo the work completed so far.

#### Example Scenario: Order Processing
Let's consider an order processing system with the following microservices:

- Order Service: Handles order creation and updates.
- Payment Service: Manages payment processing.
- Inventory Service: Manages product inventory.
#### Step-by-Step Implementation
1. Order Service:

- Accepts an order request and creates an order.
- Publishes an OrderCreated event to EventStoreDB.
2. Payment Service:

- Listens for the OrderCreated event.
- Processes the payment.
- Publishes a PaymentProcessed event to EventStoreDB.
3. Inventory Service:

- Listens for the PaymentProcessed event.
- Reserves the items in the inventory.
- Publishes an InventoryReserved event to EventStoreDB.
4. Saga Orchestrator:

- Listens for events from EventStoreDB and coordinates the saga.
- Decides the next steps based on the events received.
- Handles failures by triggering compensating transactions.
##### Implementing with EventStoreDB and PostgreSQL
1. Order Service:
```go
// Order Service
type Order struct {
    ID     string
    Status string
    Items  []OrderItem
}

func (s *OrderService) CreateOrder(order Order) error {
    // Save order to PostgreSQL
    err := s.orderRepo.Save(order)
    if err != nil {
        return err
    }
    // Publish OrderCreated event to EventStoreDB
    event := OrderCreatedEvent{OrderID: order.ID, Items: order.Items}
    return s.eventStore.AppendEvent(event)
}
```
2. Payment Service:
```go
// Payment Service
type PaymentService struct {
    eventStore EventStore
    paymentRepo PaymentRepository
}

func (s *PaymentService) ProcessPayment(orderID string, amount float64) error {
    // Process payment (e.g., call payment gateway)
    err := s.paymentRepo.Process(orderID, amount)
    if err != nil {
        return err
    }
    // Publish PaymentProcessed event to EventStoreDB
    event := PaymentProcessedEvent{OrderID: orderID}
    return s.eventStore.AppendEvent(event)
}

// Event handler for OrderCreated event
func (s *PaymentService) HandleOrderCreated(event OrderCreatedEvent) {
    // Extract order details and process payment
    orderID := event.OrderID
    amount := calculateAmount(event.Items)
    s.ProcessPayment(orderID, amount)
}
```
3. Inventory Service:
```go
// Inventory Service
type InventoryService struct {
    eventStore EventStore
    inventoryRepo InventoryRepository
}

func (s *InventoryService) ReserveInventory(orderID string, items []OrderItem) error {
    // Reserve inventory
    err := s.inventoryRepo.Reserve(orderID, items)
    if err != nil {
        return err
    }
    // Publish InventoryReserved event to EventStoreDB
    event := InventoryReservedEvent{OrderID: orderID}
    return s.eventStore.AppendEvent(event)
}

// Event handler for PaymentProcessed event
func (s *InventoryService) HandlePaymentProcessed(event PaymentProcessedEvent) {
    // Extract order details and reserve inventory
    orderID := event.OrderID
    items := s.getOrderItems(orderID)
    s.ReserveInventory(orderID, items)
}
```
4. Saga Orchestrator:
```go
// Saga Orchestrator
type SagaOrchestrator struct {
    eventStore EventStore
    orderRepo  OrderRepository
}

func (s *SagaOrchestrator) HandleEvent(event interface{}) {
    switch e := event.(type) {
    case OrderCreatedEvent:
        // Forward event to PaymentService
        s.paymentService.HandleOrderCreated(e)
    case PaymentProcessedEvent:
        // Forward event to InventoryService
        s.inventoryService.HandlePaymentProcessed(e)
    case InventoryReservedEvent:
        // Mark order as completed
        s.orderRepo.UpdateStatus(e.OrderID, "Completed")
    }
}

// Compensating actions
func (s *SagaOrchestrator) HandleFailure(event interface{}) {
    switch e := event.(type) {
    case PaymentFailedEvent:
        // Compensate by canceling the order
        s.orderRepo.UpdateStatus(e.OrderID, "Canceled")
        // Optionally publish OrderCanceled event
    case InventoryReservationFailedEvent:
        // Compensate by refunding the payment
        s.paymentService.RefundPayment(e.OrderID)
        // Update order status to "Canceled"
        s.orderRepo.UpdateStatus(e.OrderID, "Canceled")
    }
}
```

##### Notes:
- EventStoreDB: Used to store and publish events. Each service appends events to EventStoreDB, and the orchestrator listens for these events.
- PostgreSQL: Used by each service to store its state. For example, the Order Service stores order details, the Payment Service stores payment records, and the Inventory Service stores inventory reservations.
- Saga Orchestrator: Manages the workflow by listening for events and triggering the next steps or compensating actions.

This setup allows you to manage complex transactions across multiple microservices, ensuring consistency and reliability in your system.