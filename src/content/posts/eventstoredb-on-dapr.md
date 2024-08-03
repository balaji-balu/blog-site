---
title: "Eventstore on Dapr"
description: "meta description"
date: 2024-07-27T05:00:00Z
image: "/images/posts/01.jpg"
categories: ["kubernetes", "eventsdb"]
authors: ["Balaji Balasundaram"]
tags: ["kubernetes", "k8s", "dapr", "eventstoredb"]
draft: false
---

Using EventStoreDB with Dapr for implementing Event Sourcing and CQRS provides a robust solution for managing event-driven architectures. EventStoreDB is a purpose-built database for event sourcing that ensures efficient storage and retrieval of events. Here’s how you can integrate EventStoreDB with Dapr to implement Event Sourcing and CQRS.

##### Setting Up EventStoreDB
- ###### Install EventStoreDB:

You can run EventStoreDB using Docker:

```sh
docker run --name eventstore-node -it -p 2113:2113 -p 1113:1113 eventstore/eventstore:latest --insecure --run-projections=All
```
- ###### Configure Dapr to Use EventStoreDB:
 
Dapr does not natively support EventStoreDB, so you'll need to implement a custom Dapr component or use HTTP/gRPC to interact with EventStoreDB.

EventStoreDB Client Setup
You can use an EventStoreDB client library for Go to interact with the database. First, install the library:

```sh
go get github.com/EventStore/EventStore-Client-Go/v2
```

#### Integrating Dapr with EventStoreDB
Since Dapr does not natively support EventStoreDB, you can use HTTP or gRPC to interact with EventStoreDB within your Dapr services.

###### Publishing Events with Dapr
###### Order Service:

```go
package main

import (
    "context"
    "log"
    "github.com/dapr/go-sdk/client"
    "github.com/EventStore/EventStore-Client-Go/v2/esdb"
)

type OrderCreated struct {
    ID    string `json:"id"`
    Item  string `json:"item"`
    Price int    `json:"price"`
}

func main() {
    cli, err := client.NewClient()
    if err != nil {
        log.Fatalf("failed to create Dapr client: %v", err)
    }
    defer cli.Close()

    settings, err := esdb.ParseConnectionString("esdb://localhost:2113?tls=false")
    if err != nil {
        log.Fatalf("Error creating connection string: %v", err)
    }

    db, err := esdb.NewClient(settings)
    if err != nil {
        log.Fatalf("Error creating client: %v", err)
    }
    defer db.Close()

    order := &OrderCreated{
        ID:    "123",
        Item:  "Widget",
        Price: 100,
    }

    data, err := json.Marshal(order)
    if err != nil {
        log.Fatalf("Error marshaling event data: %v", err)
    }

    event := esdb.EventData{
        ContentType: esdb.ContentTypeJson,
        Data:        data,
        EventType:   "OrderCreated",
    }

    _, err = db.AppendToStream(context.Background(), "orders-123", esdb.AppendToStreamOptions{}, event)
    if err != nil {
        log.Fatalf("Error appending to stream: %v", err)
    }

    // Publish event using Dapr
    if err := cli.PublishEvent(context.Background(), "pubsub", "orders", order); err != nil {
        log.Fatalf("failed to publish event: %v", err)
    }

    log.Println("Order created, event stored, and event published")
}
```
###### Inventory Service:

```go
package main

import (
    "context"
    "log"
    "github.com/dapr/go-sdk/service/common"
    "github.com/dapr/go-sdk/service/grpc"
    "github.com/EventStore/EventStore-Client-Go/v2/esdb"
)

func main() {
    s, err := grpc.NewService(":50001")
    if err != nil {
        log.Fatalf("failed to start service: %v", err)
    }

    if err := s.AddTopicEventHandler(&common.Subscription{
        PubsubName: "pubsub",
        Topic:      "orders",
    }, eventHandler); err != nil {
        log.Fatalf("failed to add topic event handler: %v", err)
    }

    if err := s.Start(); err != nil && err != context.Canceled {
        log.Fatalf("failed to start service: %v", err)
    }
}

func eventHandler(ctx context.Context, e *common.TopicEvent) (retry bool, err error) {
    log.Printf("Order event received: %v", e.Data)

    settings, err := esdb.ParseConnectionString("esdb://localhost:2113?tls=false")
    if err != nil {
        return false, err
    }

    db, err := esdb.NewClient(settings)
    if err != nil {
        return false, err
    }
    defer db.Close()

    stream, err := db.ReadStream(context.Background(), "orders-123", esdb.ReadStreamOptions{}, 0)
    if err != nil {
        return false, err
    }
    defer stream.Close()

    for {
        event, err := stream.Recv()
        if err != nil {
            if err == esdb.ErrEOF {
                break
            }
            return false, err
        }

        log.Printf("Received event: %s", event.OriginalEvent().EventType)
        // Process the event data as needed
    }

    return false, nil
}
```
##### Summary
By integrating EventStoreDB with Dapr, you can leverage the strengths of both technologies to implement a robust event sourcing and CQRS architecture. EventStoreDB handles the efficient storage and retrieval of events, while Dapr provides a simplified way to build and manage microservices with built-in support for pub/sub messaging, state management, and more.


