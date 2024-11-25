# Transactional Outbox


## Overview

In some information exchange scenarios between systems, it is crucial to ensure certain delivery guarantees for events. The communication methods adopted within the company are described in [communication primitives](communication-primitives.md) section. In most cases, events in our infrastructure are delivered with an "at most once" guarantee.
This ADR considers ways to improve delivery guarantees to "at least once" using the Transactional Outbox architectural pattern.

This document discusses the requirements for delivery guarantees of data to the chosen transport. For example, to a message broker like Kafka. Further impact on data delivery guarantees depending on the chosen transport or implementation method of the data consumer needs to be considered separately.


## Options

To implement event capture for data changes in the service, we can use two approaches, each with its own pros and cons.


### CDC

CDC (Change Data Capture) is a methodology used in databases to track and capture changes. This allows reacting to data changes such as insertion, update, or deletion of records and performing corresponding actions based on these changes. In particular, we often use this methodology to deliver database events to the corporate data warehouse (DWH).

The advantages of this technology include:

* integration simplicity - no need for code development;
* low database load as changes are captured through binlog and the ability to handle a high stream of changes.

However, CDC has serious drawbacks:

* lack of event delivery guarantees. The maximum achievable guarantee for us is "at most once";
* violation of the service database structure encapsulation, leading to strong coupling between the source service and data consumers.


### Transactional Outbox

Transactional Outbox is a design pattern that helps ensure reliable and atomic message or event delivery in distributed systems. The main idea is to atomically add change events to a special table called the `outbox` when performing database transactions that modify any entity. Then, when the transaction successfully completes, the system should extract these messages from the outbox and guarantee to send them to destinations such as message brokers or event processing services.

The advantages of Transactional Outbox include:

* wide customization space for events triggering data transmission;
* encapsulation of the service database structure, and thus weak coupling with data consumers;
* atomicity of write operations and strict "at least once" event delivery guarantees.

The drawbacks of applying this pattern are:

* increased database load, which can lead to table locks;
* the need to customize the application for each case.

Any outbox implementation must ensure:

* atomicity: all events in the `outbox` table must either be written simultaneously with the entity state change in the database or not written at all;
* idempotence: even if the event from the `outbox` table is sent multiple times due to failures or resend attempts, its processing should not lead to undesirable effects;
* resilience: even in case of failures in the system as a whole or only in the service itself, event delivery should continue after recovery;
* consistency: the system must guarantee eventual consistency between created and sent events;
* ordering: Events in the `outbox` table MUST be stored and sent to the destination in the order of their creation concerning the work with a single instance of a domain entity.


### Decision

To form data change streams in the services, we will use both data collection options depending on the required data delivery guarantees and the expected load.


#### CDC ([Debezium](https://debezium.io/))

It must be used when fast integration with DWH is needed, and there are no strict event delivery guarantees ("at most once" is acceptable) or the expected load exceeds the thresholds acceptable for Transactional Outbox.

If the frequency of changes exceeds the Transactional Outbox thresholds but event delivery guarantees cannot be reduced, it is necessary to consult with the Architecture team and possibly redesign the solution.


#### [Transactional Outbox](https://github.com/inDriver/outbox)

In cases where strict delivery guarantees to the destination point ("at least once") are required and the event flow does not exceed a certain threshold limited by the implementation, the outbox library developed within the company must be used. We have conducted load testing of this implementation and can guarantee normal operation:

* up to 1000 EPS (events per second) when the load is evenly distributed across the business keys of the modified entities;
* up to 300 EPS when there are load imbalances and some keys are updated much more frequently than others.

It is not advisable to indiscriminately increase delivery guarantees for all events. It is always necessary to assign the minimally sufficient level to ensure optimal company infrastructure performance. The main candidates for achieving "at least once" delivery guarantees are events necessary for building GR and financial reporting.

To avoid data loss for both methods, an event queue monitoring must be implemented for sending and alerts in case of delays.
