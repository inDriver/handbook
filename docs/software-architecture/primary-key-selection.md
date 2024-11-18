# UUIDv7 Primary Key Selection


## Overview

In our system, we have traditionally utilized auto-incremented integer keys as the primary key for our database tables.

This approach has provided a simple and efficient means of ensuring unique identifiers for our records. However, as our
application has grown and our data needs have become more complex, we've encountered limitations with this approach.

We are now exploring alternative solutions that can better accommodate the requirements of quickly-generated data spread
across multiple regions and individual data-shards.


## Limitations of Auto-Incremented Keys

Auto-increment keys were previously used as a straightforward choice for primary keys in many scenarios. But they are not
suitable in cases where data generation is distributed across multiple regions or systems. In such scenarios, coordinating
and ensuring the uniqueness of auto-incremented values across distributed environments can be challenging and often impractical.

This limitation can lead to synchronization issues, data conflicts, and hinder scalability efforts listed below:

* **exposure of entity counts**: auto-increment keys inadvertently disclose the number of entities to external users, enabling them to infer business characteristics by inspecting current auto-increment field values. This can potentially reveal sensitive metrics such as daily rides, user counts, or payment transactions;
* **security vulnerabilities**: the predictability of auto-incremented identifiers can be exploited by malicious actors who may attempt attacks through incremental identifier guessing or sequentially fetching entity lists. This vulnerability aligns with the Common Weakness Enumeration (CWE) identifier [CWE-340: Generation of Predictable Numbers or Identifiers (4.12)](https://cwe.mitre.org/data/definitions/340.html)
* **primary key collisions**: in a master-master replication mode, simultaneous insertion of competing records into two
  geographically distributed copies of the database can result in primary key collisions, disrupting data integrity and consistency;
* **Data Warehousing (DWH) and Database Aggregation**: when integrating data into a Data Ware House (DWH) for Business
  Intelligence (BI) or migrating data between databases in different regions, auto-increment key problems may surface.
  These challenges include managing synchronization and maintaining data integrity.

Given these limitations, we recognize the need to explore alternative primary key strategies, such as UUIDs, to better
accommodate our geographically distributed data requirements while maintaining data privacy, security, and integrity.


## Limitations of Previous UUIDv4 Version

UUIDv4 has been selected later as a better replacement for the auto-incremented keys. UUIDv4 has successfully addressed some limitations of auto-incremented integer keys by providing a globally unique identifier without centralized coordination.

Below is the layout for the UUIDv4, which consists of truly random parts concatenated with special bits for `ver` and `var`:

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           random_a                            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          random_a             |  ver  |       random_b        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|var|                       random_c                            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           random_c                            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

However, UUIDv4 introduces new challenges:

* **ordering**: UUIDv4 is not designed for strict ordering, as its randomness can lead to suboptimal performance when
  querying ranges of data. Thus, all PK sorting optimizations were not available with UUIDv4;
* **partitioning**: while UUIDv4 helps with global uniqueness, it doesn't inherently solve partitioning issues in distributed databases, resulting in the need of adding extra `created_at` fields and indexes, that can be used for partitioning. But adding `created_at` field as partition key breaks uniqueness of the PK, now two or more records can have the same UUID, but with different `created_at` time value which breaks some fundamental constraints;
* **performance issues**: if Primary Key is random, then row should be inserted in random page on the disk, this adds an extra overhead for insert operations because clustered index should be rebalanced for each operation. There is an article [UUIDs are Popular, but Bad for Performance](https://www.percona.com/blog/uuids-are-popular-but-bad-for-performance-lets-discuss/) by Percona that describes perfectly why UUIDs (old versions, including UUIDv4) are bad for performance.

These limitations recently forced us to look for another options and investigation how can we solve this list of issues.


## Decision

After careful consideration of various options, we have decided to transition from auto-incremented integer keys, UUIDv1 and UUIDv4 formats to the
[UUIDv7 (Universally Unique Identifier version 7)](https://datatracker.ietf.org/doc/html/draft-ietf-uuidrev-rfc4122bis#name-uuid-version-7) as the primary key for our heavily used and distributed database tables. UUIDv7 draft contains a lot of explanation why previous version of UUIDs are not good in modern systems. Read more at [UUIDv7 Introduction](https://datatracker.ietf.org/doc/html/draft-peabody-dispatch-new-uuid-format-04#name-introduction-2).


## Advantages of UUIDv7

We have chosen UUIDv7 as our primary key solution due to its unique combination of benefits, making it well-suited for our use case of quickly-generated data across multiple regions:

* **Global uniqueness**: UUIDv7 generates identifiers using a combination of timestamp, namespace, and random data, ensuring global uniqueness without centralized coordination;
* **deterministic ordering**: unlike UUIDv4, UUIDv7 generates identifiers in a manner that allows for better ordering of data, making it more efficient for range queries and analytics;
* **partitioning support**: UUIDv7 introduces hierarchical structure, facilitating improved data distribution and partitioning strategies in distributed databases;
* **backward compatibility**: UUIDv7 retains compatibility with previous UUID versions, ensuring a smooth migration path from our existing UUIDv4 implementation.

Below is the layout for the UUIDv7:

```
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

As you can notice, UUIDv7 values are created by allocating a Unix timestamp in milliseconds in the most significant 48 bits, which makes such UUID sortable and partition-able, while preserving randomness between different regional instances.

Given our requirements for quickly-generated data spread across multiple regions, UUIDv7 strikes a balance between global uniqueness, ordering, and partitioning support. This choice aligns with our goal of improving system performance, scalability, and data integrity in a distributed environment.


## Database-Related Aspects of Working with UUIDv7

After careful consideration, we have decided to store UUIDv7 values as `BINARY(16)` over `VARCHAR(36)` in our MySQL databases.

This choice offers several advantages, including reduced storage overhead, faster indexing, and improved query performance for UUID-based comparisons and sorting. Storing UUIDs as binary values also ensures that they occupy a fixed amount of space in the database, regardless of their textual representation.

> **IMPORTANT**
> 
> All UUIDv7 values must be generated only on our backend side. UUIDv7 value MUST NOT be generated on client side because time on client devices can be not-in-sync, or attacker will generate own values, which can be used to bypass ordering rules or even create collisions.

To work with UUIDv7 values stored as `BINARY(16)`, we can use the MySQL functions `UUID_TO_BIN()` and `BIN_TO_UUID()` to convert between UUID representations and binary storage efficiently.

When inserting UUIDv7 values into the database, convert them to binary using UUID_TO_BIN():

```sql
INSERT INTO your_table (id, other_columns) VALUES (UUID_TO_BIN(UUIDv7_value), other_values);
```

When retrieving UUIDv7 values from the database, convert them back to UUID format using `BIN_TO_UUID()`:

```sql
SELECT BIN_TO_UUID(id) AS UUIDv7, other_columns FROM your_table WHERE ...
```


## Partitioning Data with UUIDv7

Knowing the fact that first 48 bits of the UUIDv7 represents a timestamp in ms, we can do partitioning by PK, which will be very efficient scheme, as such key grows linearly and select by order UUID will go directly to the desired partition, without scanning all partitions.

To extract a timestamp in milliseconds or datetime from the UUIDv7, following commands can be used:

```sql
SELECT CONV(HEX((_binary 0x018874410C0000000000000000000000 >> 80)), 16, 10) -- 1685577600000ms
SELECT FROM_UNIXTIME(CONV(HEX((_binary 0x018874410C0000000000000000000000 >> 80)), 16, 10) / 1000) -- 2023-06-01 00:00:00

-- For deal-history database
SELECT (FROM_UNIXTIME(CONV(HEX(`identity` >> 80), 16, 10) / 1000)) from deals partition(p2023_09) order by `identity` desc limit 1;
```

Same technique can be used later to extract virtual human-readable `created_at` DATETIME column from the UUIDv7 prefix:

```sql
 `created_at` DATETIME GENERATED ALWAYS AS (FROM_UNIXTIME(CONV(HEX((`uuid` >> 80)), 16, 10) / 1000)) VIRTUAL,
```

To create a UUIDv7 range for partitioning from known date:

```sql
SELECT UNHEX(RPAD(LPAD(HEX(UNIX_TIMESTAMP('2023-12-01 00:00:00UTC') * 1000), 12, '0'), 32, '0')) as uuidv7; -- 018c22acd00000000000000000000000
```


### Same Millisecond Issue with UUIDv7

The "same / last millisecond" issue when using UUIDv7 arises from the theoretical and practical possibility of having multiple records with the same timestamp prefix (derived from `UNIX_TIMESTAMP`) but differing random parts in the low bytes of the UUID. We see about < 0.1% of such records in our heavy databases.

In high-load environments like ours, especially when inserting new UUIDv7 values into a database, this can lead to ordering challenges when dealing with UUIDv7 keys for this 0.1% of data but still it is important to know about this issue.

In particular, when a new UUIDv7 is generated for a specific millisecond and the random part in the low bytes of this UUIDv7 is smaller than an existing UUIDv7 value already in the database for the same millisecond, it may result in the new UUIDv7 being considered "less than" the existing one. This situation can disrupt expected chronological ordering of records and potentially affect queries or data analysis that rely on the natural order of UUIDv7 values.

To mitigate this issue:

* be careful when selecting the `max(uuid)` value from the database, as new record can be technically lower for the last ms comparing to the existing value already stored in DB;
* when receiving data in realtime don't make any assumptions about ordering of the last ms of data. Just use it as stream of keys, always expect that you can receive a key for the same ms, but with smaller random part;
* for data ordering guarantee, refer to the binglog data for such events to determine exact ordering of data based on GTID. This should be our main strategy for data warehouses and business analysis;
* when using UUIDv7 as cursor, always use sorting by this key for deterministic order of data and to avoid situations when some records MAY be skipped due to incorrect assumption about sorting of data in the storage.

Below is real example of orders from the deal-history (`latam-br` region), which have occurred in the same ms:

```
2023-09-15 23:01:01 | 018a9b13-8277-716a-9e51-f0da4e4d494e
2023-09-15 23:01:01 | 018a9b13-8277-716b-a90c-913b8759bde1
```
And there aren't any guarantees about ordering for such events, this is why additional attention and handling may be required.
