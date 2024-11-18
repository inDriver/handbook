# inDrive Universal Entity Identifier


## Overview

inDrive has employed various formats for generating universal entity identifiers to uniquely identify entities across distributed systems. Initially, auto-incremented identifiers were used, which served their purpose but presented limitations, especially in distributed environments. To address the need for globally unique identifiers, inDrive transitioned to UUIDv4. However, the randomness of UUIDv4 negatively affected database performance, particularly in operations like partition pruning and inserts.

In response, a decision was made to migrate to UUIDv7, which leverages a monotonic timestamp prefix to improve database performance and enable more efficient partitioning.

See [primary key selection](primary-key-selection.md) for additional context.


## Advantages of UUIDv7

UUIDv7 have previously chosen as our primary key solution due to its unique combination of benefits, making it well-suited for use case of quickly-generated data across multiple regions:

* **global uniqueness**: UUIDv7 generates identifiers using a combination of timestamp and random data, ensuring global uniqueness without centralized coordination;
* **deterministic ordering**: UUIDv7 generates identifiers in a manner that allows for better ordering of data, making it more efficient for range queries and analytics;
* **partitioning support**: UUIDv7 introduces hierarchical structure, facilitating improved data distribution and partitioning strategies in distributed databases.

Below is bit-layout for the UUIDv7:

```text
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           unix_ts_ms                          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          unix_ts_ms           |  ver  |       rand_a          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|var|                        rand_b                             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                            rand_b                             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

UUIDv7 values are created by allocating a Unix timestamp in milliseconds in the most significant 48 bits, which makes such UUID sortable and partition-able, while preserving randomness between different regional instances.


## Limitations of Existing UUIDv7 Version

While UUIDv7 improved database operations, it introduced challenges in identifying the regional database associated with a specific entity identifier, making it difficult for support teams to quickly locate the correct regional database instance:

* **origin of entity**: unique identifier solves uniqueness, but how to find exact entity in any regional database without prior knowledge of entity's country? Or how to show global history of rides or orders around the world?;
* **routing of requests**: how to send cross-regional requests if we know only identifier of entity to proper regional instance? UUIDv7 does not contain any information, which might be used for routing purposes. We would like to have peering model between regional services to provide universal entry-point for our users instead of regional instances of services (p2p / mesh of regional services);
* **sharding support**: UUIDv7 gives us working partitioning scheme, but we also want to have sharding scheme on top of what we have right now.


### Decision


#### Identifier Requirements

Our decision based on our own findings and observations, and detailed review of similar known universal identity implementations, used in projects / companies:

* [LexicalUUID] by Twitter;
* [Snowflake] by Twitter;
* [Flake] by Boundary;
* [ShardingID] by Instagram;
* [KSUID] by Segment;
* [Elasticflake] by P. Pearcy;
* [FlakeID] by T. Pawlak;
* [Sonyflake] by Sony;
* [orderedUuid] by IT. Cabrera;
* [COMBGUID] by R. Tallent;
* [ULID] by A. Feerasta;
* [SID] by A. Chilton;
* [pushID] by Google;
* [XID] by O. Poitrey;
* [ObjectID] by MongoDB;
* [CUID] by E. Elliott.

We identified the set of requirements that should be satisfied in our new standard:

* identifier must be k-sortable. That is, values within or close to the same timestamp are ordered properly by sorting algorithms. Be aware about [same 1ms issue in the UUIDv7][UUIDv7 Same ms issue];
* identifier should be big-endian with the most-significant bits of the time embedded as-is without reordering;
* identifier should utilize millisecond precision and Unix Epoch as timestamp source;
* identifier format should be Lexicographically sortable while in the textual representation;
* identifier must not require unique network identifiers as part of achieving uniqueness;
* distributed nodes must be able to create collision resistant Unique IDs without a consulting a centralized resource;
* identifier must contain unique identifier of country where such entity has been created;
* identifier should contain unique identifier of service where such entity has been created.

While the initial requirements were inherited from UUIDv7, two additional requirements were introduced to support data sharding schemes. For instance, if we know the entity’s country and the service name, we can perform a lookup in our global routing table within the Router component to determine the exact node and region responsible for processing that entity.


#### UUIDv8 Base Format

Considering all these requirements, we determined that our entity identifier must be based on the UUIDv8 format. UUIDv8 provides an RFC-compatible format suitable for experimental or vendor-specific use cases, with the only requirement being that the `variant` and `version` bits must be set. This structure allows UUIDv8 the unique flexibility to incorporate custom fields within the identifier, and it has the following bit layout:

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           custom_a                            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          custom_a             |  ver  |       custom_b        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|var|                       custom_c                            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           custom_c                            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

This flexible UUIDv8's bit layout enabling to organize layout, incorporating additional meaningful bits of data to create an enhanced context for partitioning / sharding / routing purposes:

* `custom_a`: the first 48 bits of the layout that can be filled as an implementation sees fit;
* `ver`: the 4 bit version field, set to 0b1000 (8);
* `custom_b`: 12 more bits of the layout that can be filled as an implementation sees fit;
* `var`: The 2 bit variant field, set to 0b10;
* `custom_c`: The final 62 bits of the layout to be filled as an implementation sees fit.


#### inDrive Universal Entity Identifier (aka iUID)

Since UUIDv8 provides us with 48+12+62=122 bits for customization, we decided to incorporate the best aspect of UUIDv7: the timestamp prefix as the first part of the identifier. This approach makes our identifier both binary and lexically sortable, which also addresses database partitioning needs. By defining binary ranges for specific time periods, we can efficiently partition data within the database.

Decision has been made to allocate `custom_b` field to the identifier of country and our internal version id.

4 bits of `ver1` must encode our own version of inDrive identifier to have an ability to extend this format in the future, should be set to 0b0000 (0) now.

8 bits of `country` field must store the `country_id` of entity, see our `tbl_country` table for the reference.

General decision has been made that `custom_c` should be split into 2 parts: `service_entity` 8 bits encoding the unique value of service's entity and all remaining bits will be allocated to the `random_a` value, giving us 54 bits of randomness, which should be sufficient from the security point of view.

8 bits of `service_entity` field must store the unique identifier of the entity type inside service. List of reserved values for this field should be added to the current document. Each key inside this enumeration should use following parts: service name + entity name inside this service. 

Example: `AyanOrder`, `AyanRide`, `IntercityOrder`, `WalletAccount`, etc.

```
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           unix_ts_ms                          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          unix_ts_ms           |  ver  |  ver1 |    country    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|var|service_entity |       rand_a                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           rand_a                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

#### Advantages of inDrive Universal Entity Identifier (iUID)

* **enhanced database performance**: the utilization of a timestamp prefix preserves the performance improvements achieved with UUIDv7, ensuring efficient database operations, including partition pruning and inserts;
* **origin of entity**: by incorporating fields for country and service identifiers within the identifier payload, iUID facilitates quick identification of the regional database associated with a given entity identifier. This simplifies support efforts, enabling faster issue resolution and enhancing customer satisfaction;
* **one identifier to rule everything**: it is primary key, it is partition key, it is sharding key as well;
* **future-proofing**: iUID's extensible format provides flexibility for accommodating future architectural requirements and evolving business needs, ensuring scalability and adaptability over time;
* **routing-friendly**: information about country of entity helps to use information from our `Router` component to find out the endpoint (node), which is responsible for this entity.


#### Disadvantages of inDrive Universal Entity Identifier (iUID)

* **less randomness**: by allocating 20 random bits from the UUIDv7 into our own meaningful fields, we reduce amount of random bits from 74 to 54. But still, probability of having collision is relatively small;
* **limited amount of unique services**: having only 8 bits (255 unique values) for the service field may limit us with number of entities which might be geo-distributed. But according to our own calculations we have only tens of entities that might benefit from geo-distribution, remaining part can work on UUIDv7 if we don't need sharding/routing
  for all entities;
* **complex bit layout**: Having a lot of small fields forces us to do bit-level work to properly fill all relevant fields. But this complexity should be covered by our own `uuid` library, which will contain working implementation.


### Future Development

Once, the iUID will be adopted and used actively in services, we may create new solutions on top of that:

* transparent routing / proxying of requests between regional instances: no more 410 errors on client-side;
* mesh-like or p2p Architecture of services with higher level of availability and decentralization;
* centralized admin / moderator panels, which will be able to route query for given entity immediately to the regional instance;
* simplified geo-distribution of global services like balance-api or external-sender.
