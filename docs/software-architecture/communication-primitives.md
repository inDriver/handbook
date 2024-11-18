# Communication Patterns


## Overview

inDrive services use various methods to communicate with each other. Broadly, all communication patterns can be divided into two major groups: synchronous and asynchronous. Modern distributed system architectures follow the principle of "smart endpoints, dumb pipes". This means that data exchange tools should act solely as transport mechanisms and should not implement any business logic. This document aims to clarify which communication patterns are appropriate for different use cases.


## Options


### Event Driven

The meaning of term `event-driven architecture` is well described by Martin Fowler in his [article](https://martinfowler.com/articles/201701-event-driven.html).


### State vs Event

A `state` is the current representation of a business entity, essentially a snapshot of data at a specific point in time. A state must have a unique identifier and a sequential value to determine the order of state changes. Each new state fully replaces the previous one, simplifying the update process.

An `event` represents a change in state. Events must have their own identifiers, a sequential value to define the event order, and may contain details of the change. The current representation of an entity can be restored by applying all events from its creation to the initial state. A special type of event is a `command`.

In most cases events should be ordered to restore a state correctly, but in some cases events may have a [commutative](https://en.wikipedia.org/wiki/Commutative_property) or [associative](https://en.wikipedia.org/wiki/Associative_property) property that makes state restoration
algorithm simpler. Each operation of applying an event to a state MUST be [idempotent](https://en.wikipedia.org/wiki/Idempotence).

This example illustrates what is state and event and why event order matters. User cannot withdraw requested funds before deposit operation completes. A time column in this example acts as a sequential value used to restore events order:

* state of a business entity "User balance" { id=1, ownerId=4, amount=10, time=1689316666 };
* event "Deposit" { uniqueKey=87, ownerId=4, time=1689317777, amount=5 };
* event "Withdraw" { uniqueKey=23, ownerId=4, time=1689318888, amount=13 };
* new state of a "User balance" entity { id=1, ownerId=4, amount=2, time=1689319999 }.

Implementing event or state data stream pros and cons:

|       | **State**                                                                                                    | **Event**                                                                                                                             |
|-------|--------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------|
| **Pros** | - simple to implement<br> - no need for resynchronization mechanisms<br> - resilient in case of message loss | - optimized for transportation                                                                                                        |
| **Cons** | - may require high bandwidth                                                                                 | - requires data change tracking mechanisms<br> - requires resynchronization in case of data loss<br> - often requires strong ordering |


### Synchronous vs Asynchronous

When choosing between synchronous and asynchronous communication, a developer should consider two key questions: “What is the data channel’s dimension—one-to-one or one-to-many?” and “Is a response required?”

For the first question, a one-to-many dimension is generally preferable, as it makes the communication channel reusable. This approach is ideal for state and event streaming, where the producer doesn’t need feedback on the operation’s result.

If the answer to the second question is yes and the dimension is one-to-one, then synchronous communication is the most suitable. In other cases, a developer should consider asynchronous communication, accounting for the added complexity of managing timeouts and potential errors.


### Decision

Synchronous communication requires additional effort to ensure that the recipient successfully receives the request. Implementing backpressure and retry logic correctly can be challenging. The longer the synchronous call chain between services, the higher the likelihood that a failure in any link will lead to a breakdown of the entire chain.

Asynchronous communication, on the other hand, typically follows a "fire and forget" principle. The message is sent, and the consumer will process it whenever it becomes available to do so.

In summary, as a general rule, developers should minimize the use of synchronous communication between services.


#### Synchronous HTTP

A developer should use HTTP REST for external public and mobile application APIs because :

* it is de-facto an industry standard;
* simple and easy to use, has a robust support by languages and frameworks;
* it's text serialization is human-readable and easy to debug at client side;
* supports convenient authentication mechanisms like JWT;
* such APIs take advantage of caching mechanisms to reduce server load.

The drawback of HTTP REST is one way nature that affects mobile applications responsiveness. Mobile clients asking for state updates may generate up to 70% of all requests to backend. This is not very efficient, so in the future we must consider switching to [SSE](https://medium.com/yemeksepeti-teknoloji/what-is-server-sent-events-sse-and-how-to-implement-it-904938bffd73) or even [QUIC](https://peering.google.com/#/learn-more/quic) protocol to build a reliable two-way communication channel to reduce latencies and server load at the same time. Still, this is the best solution to build public APIs.

Internal APIs for frontend should use Graphql because:

* it solves a problem of data overfetching at low cost;
* introduces a strong, self-documenting schema that describes available types and operations;
* allows clients to fetch all required data with a single request;
* eliminates the need of adding a BFF.

Graphql must not be used for public APIs due to a vast number of vulnerabilities and a complex way to build data authorization mechanisms.


#### Synchronous gRPC

A developer should choose gRPC to implement inter-service communications at backend. gRPC offers such benefits as:

* higher performance in terms of latency compared to REST;
* language-agnostic service and message schema description with strong typing;
* a compact binary serialization, leading to lower bandwidth usage and faster serialization / deserialization;
* bidirectional streaming allows building two-way communications and reduce number of polling requests.

However, gRPC APIs have several caveats that may have a significant impact on a service performance:

* gRPC clients open persistent connections to target resources that requires special handling on a client side to perform a correct load balancing;
* it needs special extensions to build authentication;
* with binary serialization it may be challenging to debug such APIs.


#### Asynchronous Transport

When choosing between Kafka and NATS (JetStream) as transport options, Apache Kafka must be the preferred choice for both internal and external event streams. This preference is driven by Kafka’s maturity as a technology and our extensive operational experience with it.

However, NATS may be selected for specific cases of internal communication and event buffering. If opting for NATS, developers should carefully consider its configuration options: in-memory vs. persistent storage and the delivery semantics it supports (`at-most-once` or `at-least-once`). When using in-memory storage, it’s essential to estimate memory consumption accurately and account for potential event production spikes to avoid performance issues.


#### Asynchronous State

Each service we design must [own its data and logic](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/architect-microservice-container-applications/data-sovereignty-per-microservice) and control at least one business entity. A service must publish this business entity changes in a form of State to a message broker with `at-least-once` delivery semantic. Loosing messages at this point is not an option, so a Transactional outbox pattern must be applied to producer. This makes integrations with any existing and future consumers much easier.

If a service does not have consumers of its State updates currently, it still must publish update messages for Data Warehouse.

State streaming assumes external data consumers, so the only suitable solution here is to use Apache Kafka transport.


#### Asynchronous Event

In addition to state streaming, a service may produce event streams for internal and external purposes with an appropriate transport.
