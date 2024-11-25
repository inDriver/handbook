# Data Delivery Guarantees


## Overview

Our distributed system, built on Kubernetes, GoLang microservices, Apache Kafka, Redis, and MySQL / Aurora, operates globally with high availability and fault tolerance. We need well-defined data delivery guarantees for both producers and consumers, aligned with a tiered service model.


## Event-Driven Approach

In our company, we rely on an event-driven architecture to facilitate seamless communication between microservices. This approach promotes loose coupling, scalability, and real-time responsiveness, enabling us to efficiently handle complex workflows and dynamic data processing requirements. By decoupling components through asynchronous messaging, we enhance system flexibility and agility, allowing services to evolve independently without disrupting the overall architecture.


## Mapping Data Streams to Levels of Guarantees

While an event-driven approach provides loose coupling, we must remain aware of potential issues such as message loss, delays, or consumer lag. These limitations require us to carefully map each data stream to an appropriate level of delivery guarantee based on its characteristics and criticality:

* business impact: different data streams may have varying levels of criticality to the business. Services handling
  mission-critical operations or sensitive financial transactions require higher delivery guarantees to ensure data integrity and regulatory compliance;
* performance requirements: adjusting delivery guarantees to match the performance requirements of each data stream
  optimizes resource utilization and minimizes latency. In some cases, this requirement will force us to prevent
  data replication because of significant amount of data to be transferred, or to comply with SLA;
* error handling complexity: complex error handling mechanisms, such as data deduplication and idempotency, may incur additional overhead and complexity, which is not needed by default in all places. Mapping data streams to appropriate
  delivery levels helps streamline development efforts and resource allocation, ensuring optimal system performance;
* operational overheads: implementing excessive delivery guarantees across all data streams can result in unnecessary
  operational overheads, impacting scalability and cost-effectiveness. We would like to have a balance between
  reliability and operational efficiency;
* regulatory compliance: compliance requirements, such as GDPR / PCI-DSS or HIPAA, may dictate the level of data integrity
  and security needed for certain streams. Adhering to regulatory standards necessitates careful consideration of
  delivery guarantees to safeguard sensitive information and mitigate legal risks.


## Decision

We decided to use a tiered service model with different levels of data delivery guarantees, ranging from the best-effort
level or "at-most-once" (default) up to the exactly-once delivery guarantees. See relevant ADR regarding service tiering.

This ADR provides a structured approach to align data delivery guarantees with the [Tiered Service Model](./service-tiering.md) in our distributed system, considering the unique requirements and challenges of each level.


## Best-Effort Delivery (Default Level)

**Description**: provides no guarantee regarding the number of deliveries. Messages or data may be lost or delivered multiple times. This is the basic level in inDrive, sufficient for most cases without additional coding or architectural improvements. It may be used by default for all new services where the risk of missing some events is acceptable—for example, a single location within a stream of user geolocations. However, it must not be used for data streams within a critical path. Developers should be aware of the limited guarantees at this level and reflect these limitations in their SLAs.

**Context**: best-effort delivery is the default as it strikes a balance between reliability and performance. It should be sufficient for services where the risk of missing some individual events is acceptable, and its simplicity reduces the need for additional coding and architectural enhancements. This level is recommended as a starting point for new services and data streams where there are no specific requirements for high-level delivery guarantees.

**Cost**: implementation cost is minimal for this level, just install and use kafka package as main dependency for producing messages. No extra tuning of settings will be required. Under the hood, [franz-go package](https://github.com/twmb/franz-go) is used, which covers low-level aspects of communication with Kafka cluster.


## At-Least-Once Delivery Guarantee

**Description**: ensures that a message is delivered to the consumer at least once. Service must ensure that data or
message has been delivered before proceeding to the next packet or message.

**Mechanism**: producer should retry sending the message until an acknowledgment is received from the messaging system.
Service may use the transactional outbox to ensure that data has been delivered at least once. Duplicate messages may occur, so consumers working with such data streams should implement data-deduplication techniques, for example, based on idempotency keys or unique identifiers of
each record.

**Context**: this level of guarantee should be used in services where each message must be delivered at least once to
ensure that all messages are processed, and none are lost.

**Cost**: implementation cost is medium, as application should adopt the usage of transactional outbox. This requires
implementation of async component, additional logic of publishing events via outbox. Pro-active monitoring of outbox
metrics should be used to avoid replication lags or issues with performance of outbox.


## Exactly-Once Delivery

**Description**: ensures that a message is delivered exactly once to the consumer, eliminating duplicate deliveries. This level of guarantee is required in scenarios where all data must be reliably transferred from one location to another. Examples include delivering a CDC (Change Data Capture) stream from the database to the data warehouse (DWH) or streaming events from Kafka to S3 for cold storage.

**Mechanism**: both the producer and consumer must use coordination, typically involving unique sequential identifiers and transactional mechanisms. High-level tools and solutions, such as replication streams, may be utilized. Generally, this level of guarantee should not be implemented within individual services due to the substantial effort required to maintain such a solution.

**Context**: primarily used in established solutions such as Kafka Connect, AWS DMS, Clickhouse pipelining, Debezium, and similar tools, as achieving exactly-once delivery requires a higher level of complexity and effort.

**Cost**: due to the high complexity and associated implementation costs, the Architecture Team has decided not to use this level of guarantee within our services. Only well-known enterprise solutions that support this guarantee must be used instead.


## Overview of Delivery Guarantees

|                           | Best-effort delivery (Tier-1 Services)                                                              | At-least-once delivery guarantee                                                                | Exactly-once delivery (high-level)                                                                                |
|---------------------------|-----------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| **Complexity**            | Relatively simple implementation with minimal overhead                                              | Moderate complexity due to handling retries and acknowledgments                                 | High complexity; significant effort required for coordination                                                     |
| **Reliability guarantee** | No guarantee regarding the number of deliveries; messages might be lost or delivered multiple times | Ensures that each message is delivered to the consumer at least once                            | Guarantees that each message is delivered exactly once, eliminating duplicates                                    |
| **Use case**              | Suitable for general services except those with critical priority                                   | Recommended for services where each message must be processed, ensuring none are lost           | Reserved for critical services where exactly-once delivery is essential                                           |
| **Implementation**        | Default level in the company, sufficient for most cases without additional coding or architecture   | Recommended to use a transactional outbox to ensure at-least-once delivery                      | Generally not implemented in individual services due to the substantial effort required for automated maintenance |
| **Data deduplication**    | Not enforced; additional effort may be needed for deduplication if required.                        | Duplicates may occur, necessitating deduplication techniques and / or idempotency keys.         | Deduplication is implicit in exactly-once delivery; typically handled by the messaging system                     |
| **Possible Lag (SLA)**    | Best-effort mode, up to 1 day of delay (refer to Tiering ADR for specifics)                         | Delay ranges from 72 minutes / day to 1 minute / day (refer to Tier-2 and Tier-3 requirements). | Minimal lag, up to 1 minute / day, as delays are minimized (see Tier-3 and Tier-4 requirements)                   |
