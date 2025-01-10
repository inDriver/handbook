# Cold / Warm Storage for Services


## Overview

At present, our company is experiencing a trend where we accumulate large volumes of data in our storages such as Aurora Databases or AWS S3. Some tables in Aurora (MySQL) Databases, for example, can easily reach sizes of several billion records and volumes between 1TB…128TB. This imposes limitations on performance of such Databases and our ability to modify such storages which requires careful handling of DDL SQL queries. Also, there are fundamental questions about how data is stored and accessed in typical queries and during periodical audit of data and processes around such data.

To address such limitations and questions, we should introduce a clear separation of data into common groups and define patterns and processes around each type of data.


## Decision


### Data Types Classification

inDrive has decided to classify all data inside storages into three main types, based exclusively on access pattern:

* **hot data**: actively written and frequently read data that requires immediate access and high availability even at a higher cost;
* **warm data**: data with limited amount of changes during period of time, but still accessed periodically, especially for auditing, analytics and reporting purposes;
* **cold data**: rarely accessed data with no modifications, stored for long-term retention at a lower cost.

To optimize our data management, we have also decided to align each data type (Hot, Warm, Cold) with appropriate storage solutions, based on storage characteristics, access patterns, and the amount of data to ensure that our storage strategy is both cost-effective and performant.


### Hot Data Storage

Hot Data represents information that is frequently accessed and modified (up to tens of thousands of QPS), often in real-time, with strict requirements on response time (up to hundreds ms), consistency, and availability. These datasets are critical to ongoing business operations, where performance and low-latency access are required. Hot data storage systems must support high throughput, handle significant read / write volumes, and ensure strong transactional guarantees, especially in mission-critical systems like our Tier-3 and Tier-4 services.

See main requirements below:

* **performance**: hot data requires low-latency access and high write performance to support real-time operations;
* **scalability**: as the volume of data grows, the storage solution must scale efficiently without sacrificing performance;
* **consistency**: strong consistency models are essential, ensuring that each transaction reflects the most up-to-date state of the system.

We decided to use traditional OLTP Storage Solutions for Hot Data type, primarily relying on MySQL and its AWS-managed equivalent, Amazon Aurora. While MySQL / Amazon Aurora databases are well-suited for high-performance OLTP workloads, they come with specific requirements to maintain optimal performance:

* **limited storage volume**: hot data should generally be kept within several terabytes of data;
* **data partitioning**: in case of high load and quick growth of such databases, effective data partitioning scheme based on time-based UUIDs (like UUIDv7) must be used to ensure manageability and ability to offload data into Warm and Cold Data Storages (see below) and improve queries performance;
* **[Partition pruning](https://dev.mysql.com/doc/refman/8.0/en/partitioning-pruning.html)** for all queries should be used to limit the scan or updates to relevant data only, reducing query time and resource usage;
* **optimized queries**: SQL queries must be optimized to leverage indexes and partitions effectively, avoiding full table scans on large datasets.

While MySQL and Aurora have been adopted as our primary solutions for working with Hot Data types, we are not limited only to them, as any other OLTP storages, directly approved in our Technology Radar can be used as a Hot Data Storage as well. If any other Hot Data Storage type is considered necessary, consultation with the Architect, Infrastructure, and DBA teams is required to extend existing set of technologies inside our Technology Radar and to ensure compatibility with our performance standards and infrastructure strategy.


#### Warm Data Storage

Warm data, which changes infrequently but still requires read access, should be stored in a manner that balances cost-efficiency with performance. Unlike Hot Data, Warm Data type usually requires several times more data volumes, typically ranging from several terabytes to tens of terabytes. Thus, Warm Data Storage must ensure that such Warm Data retrieval is quick and predictable, without performance degradation as data volume increases up to tens of terabytes of data.

To manage warm data effectively, we have the following options:

* **Partitioning in Existing OLTP Hot Data Storage**: Warm data may be separated inside existing primary OLTP Hot Data Storage like MySQL or Aurora by organizing data inside it into time-based partitions within the same database. Data inside tables MUST be partitioned then by Primary Key based on the [primary key selection](primary-key-selection.md) or [universal entity identifier](universal-entity-identifier.md) format, containing encoded information about timestamp inside keys and partitions. Recent (or last) partitions can be designated then as Hot Data partitions, while older partitions may slowly cool into Warm or even Cold Data Partitions. This approach should be used as a primary one, allowing for efficient management of data while leveraging existing infrastructure and does not require additional components or changes inside applications assuming that all necessary database best-practices have been applied properly;
* **Offloading to OLAP Warm Data Storage**: alternatively, warm data MAY be offloaded to an OLAP storage solution, such as ClickHouse or any other approved OLAP solutions in our Technology Radar. The cost of such OLAP Data Storage must also be taken into account and should be minimized. OLAP storages are designed to handle large volumes of data with complex queries efficiently. They provide rapid read access and maintain performance even as data grows, making them suitable for analyzing and querying larger datasets without linear performance degradation. OLAP Warm Data Storage should support infrequent updates and modification of previously offloaded data. This is because some change operations may still occur, which could result in different checksums or eventual inconsistencies between primary OLTP Hot Data Storage and secondary OLAP Warm Data Storage. If the chosen type of Data Storage has limitations on the number of simultaneous requests (e.g. tens...hundreds of RPS) or operates with high latency (more than 500ms...1s), the team must implement asynchronous request processing both in the application's UI and on the backend side, for example by implementing an internal request queue with a timeout and a circuit breaker in case when storage is unavailable or busy. This approach should be used if price of organizing OLAP storage is cheaper comparing to the price of organizing warm partitions inside primary Hot Data Storage (see the first option) and such service receives very limited amount of requests, suitable for OLAP storage (usually for different admin backends, analytic purposes and similar). Must not be used as a source for any financial/critical reporting Data Storages without defined Data Quality checks.
- **Offloading Data into a Separate Service**: Warm data may be also offloaded into a separate history service by communication channels providing sufficient Data Delivery guarantee level, described in the [data delivery guarantees](data-delivery-guarantees.md) and storing such data in a separate relational database. Such history service should have its own API for data access - proxying requests through the main service is not recommended (except when it is necessary to combine query results from both Hot and Warm storage). Access frequency to the history service should be relatively low - tens of RPS, and its data should not be used in any critical processes inside Tier-3/Tier-4 services. If all these conditions are met, the stability requirements for the new history service may be lower than those of the original one because access pattern will be completely different. Separate Service with own Data Storage must not be used as a source for any financial / critical reporting Data Storages without defined Data Quality checks.


#### Cold Data Storage

Cold Data Storage has been accepted and recognized as reliable and inexpensive storage solution with limited or restricted possibility of immediate access and almost unlimited storage volume. It is assumed that read access to such Cold Data should not exceed several times per year. Cold Data must not be updated or changed in any processes. To store such Cold Data, any reliable and cheap storages, described in our Technical Radar may be used.

Currently, Amazon S3 has been chosen as the company’s primary Cold Data Storage solution due to its strong reliability, availability, and compliance features, essential for meeting our financial, long-term data retention and audit requirements:

* **durability and availability**: S3 provides 99.999999999% durability by replicating data across multiple geographically dispersed Availability Zones (AZs), ensuring near-perfect data preservation from the point of Data Loss protection. With sufficient level of availability more than 99.9%, see [AWS S3 - SLA](https://aws.amazon.com/ru/s3/sla/), AWS S3 guarantees that our data remains accessible;
* **data integrity and auditability**: S3 ensures data integrity through automatic checks and repairs using checksums. The service also supports versioning and Object Lock, allowing data to be stored in a write-once, read-many (WORM) mode. This ensures that data cannot be altered or deleted during a specified retention period, which is crucial for compliance with regulations like SEC Rule 17a-4(f) or similar.
* **compliance and security**: S3 offers robust encryption (both server-side and client-side) and holds a range of compliance certifications, including GDPR, HIPAA, and SOC 1, 2, and 3. These features ensure that our Data stored in S3 meets stringent security and privacy standards.

> **NOTE**
>
> Personal data MUST not be stored in any Cold Data Storages without defined and limited retention period to ensure that we have the ability to modify or remove such data according to legal requirements and our User Termination processes.
> Personal data MAY be stored only as an encrypted backup for short period of time, see Backup Retention time in our Backup Policy for additional information.


#### Data Deletion Requirements

For data used in financial or critical reporting, strict safeguards must be in place to protect Data Storage against accidental DELETE or DROP requests. Access rights must be configured to prevent accidental deletion. Services handling such data should be designed with an append-only approach, avoiding any explicit or implicit data deletion or even updates. This ensures data integrity and traceability over time.

Instead of deleting data, a formal data offloading process must be used (see below), accompanied by partitioning strategies to maintain the database's performance and manageability. This approach allows old or less frequently accessed data to be safely moved to appropriate Warm/Cold Data Storage tiers while preserving the integrity of such data.

> **NOTE**
>
> For services whose data is directly used in financial reporting and auditing, at least one full previous year of data MUST be stored inside Primary Data Storage, no matter if Data is Warm or Cold already. This is necessary to accommodate read operations by audit companies and our internal data quality processes.
>
> Once full set of ITGC-controls will be active, this requirement MAY be corrected.

For services where Warm or Hot data storage contains Primary Personal Information (PPI) and data is stored in the service for more than 6 months, integration with the User Termination process must be added. In these cases, DELETE queries may be used explicitly to remove PPI or to patch records to erase only the specific portions containing personal information. This ensures compliance with privacy regulations while maintaining the overall integrity of the data.


#### Data Offloading Guarantees

When offloading Hot, Warm or Cold Data from the primary Data Storage, we focus on ensuring that the data maintains its quality across different data storages. Regardless of the pattern, all data from the Primary Data Storage MUST be offloaded to secondary storages (as OLAP Data Storages, S3 Storage, etc) with a sufficient level of Data Delivery guarantees, described in the [data delivery guarantees](data-delivery-guarantees.md).

> **NOTE**
>
> Additional restrictions on data offloading are imposed if [personal information handling](personal-information-handling.md) is required.

While Data Delivery Guarantees gives us knowledge about the fact that each change from the primary storage has been delivered with at-least-once or exactly-once level of guarantees, we MUST also run automated Data Quality process to ensure that offloaded data meets our own criteria.

For Hot and Warm data, the following Data Quality characteristics must be achieved:

* **completeness**: we should verify completeness by comparing the number of unique UUIDs (primary keys) in the primary storage with those in the secondary storage over the past N-1 time interval (from minutes to a day). The counts MUST match, indicating that all records have been accurately offloaded. Records in the secondary storage MAY be missing in case of insufficient `best-effort`/`at-most-once` Data Delivery guarantee levels or technical problems.
* **uniqueness**: we should ensure uniqueness by checking that the count of total records within the N-1 time interval matches between the primary and secondary storage. There MUST be no duplicate records in the secondary storage. Duplicate records in the secondary storage MAY happen if `at-least-once` Data Delivery guarantee level has been used without data deduplication/data merge approaches.
* **freshness**: freshness should be checked either by comparing delta of incremental value of PK in primary and secondary storages, or by comparing the timestamp derived from the latest [primary key selection](primary-key-selection.md) or [universal entity identifier](universal-entity-identifier.md) PK in the primary storage with corresponding value in the secondary storage. By extracting and comparing these values or timestamps, we can determine the lag (or age) of the data and ensure that it reflects the most recent updates.

> **NOTE**
>
> For Hot and Warm data offloading specifically, it is important to note that consistency cannot be guaranteed over time due to the potential for eventual consistency issues. Changes to warm data can occur throughout the storage period, and thus, absolute consistency at any given moment cannot be assured.

In addition to the quality characteristics outlined for offloading hot and warm data, the following additional guarantees when offloading cold data must be achieved:

* **immutability**: since cold data is no longer subject to changes, we must enforce immutability by prohibiting any modifications to these records in both the primary and secondary storage systems. From the moment data is classified as cold, it MUST be accessible in a read-only mode, ensuring its integrity over time.
* **data consistency**: with no further changes occurring in cold data, we must perform comprehensive verification processes (see below) to ensure consistency between the primary and secondary storage. This includes comparing checksums, modification dates, and the exact number of records, providing auditable level of confidence in the 100% accuracy and completeness of offloaded data across storage systems. Such artifacts must be stored near offloaded data to prove data consistency level.


#### Cold Data Offloading Process

To ensure the Data Consistency and integrity of Cold data, the following process should be implemented:

* data tables requiring cold storage must use an approved architectural data schemes, including UUIDv7 / UUIDv8 primary keys and partitioning by primary key. At the beginning of each period (e.g., monthly), a new partition must be created;
* life-cycle policy should be attached to such data to know the required retention periods of hot and warm data respectively. Each service may have own Life-Cycle policy, describing how much data is considered as Hot, Warm and respectively Cold: e.g. for orders we have 1 month of Hot data and 2 more months of Warm data, all remaining data will be Cold. In this case partitions should be retained at least for three months + 1 current active partition, and the oldest four-month-old partition may be unloaded if it isn't violate any previously mentioned requirements. For data that will no longer change (cold data), a data offloading process may be initiated to transfer this data into Cold Data Storage;
* only the oldest partition of data may be atomically detached from the primary table using the `ALTER TABLE partitioned_source EXCHANGE PARTITION pXXXX_YY WITH TABLE source_pXXXX_YY` [Partition Exchange Operation](https://dev.mysql.com/doc/refman/8.0/en/partitioning-management-exchange.html) and moved to a dedicated table with the same scheme, with write permissions revoked. Once detached, this data becomes fully immutable since it is no longer part of the main table and cannot be modified by applications or operators;
* this detached partition MUST then be exported to Amazon S3 as an object according to the [Exporting DB cluster data to Amazon S3 - Amazon Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/export-cluster-data.html) guidelines. The export task must be monitored to ensure success, with any errors reported through alerting systems, leading to an investigation;
* such exported partition must be again restored from the AWS S3 back into the database under a different name (`source_pXXXX_YY_restored`). Hash (or checksum) comparison must be performed between the original detached partition and the restored partition from S3. If the hashes match, consistency between the detached partition and the cold storage is confirmed;
* once consistency is confirmed, the WORM (Write Once, Read Many) lock MUST be applied to the S3 object, preventing any modifications and allowing only read access. All verification artifacts, including timestamps, verification results, table names, and data checksums, must be also saved in S3 alongside the partition and protected with a WORM lock. These artifacts MUST be retained for the required audit period (up to 7 years);
- finally, once data is safely and consistently stored inside Cold Storage, the restored partition table and detached partition itself should be deleted, leaving a verified, consistent and immutable copy in S3. This process ensures that cold data is consistent, protected from modification, and verifiable for audit purposes.


#### Data Type Comparison

Below is short summary of different data types and their characteristics:

| Type of data** | Write operations                 | Read operations                   | Data volume**    | Data storage type     | Storage cost |
|----------------|----------------------------------|-----------------------------------|------------------|-----------------------|--------------|
| Hot data       | Constant, up to thousands of QPS | Constant, up to thousands of QPS  | Up to several TB | Primarily OLTP        | Expensive    |
| Warm data      | Rare, individual requests        | Occasional, up to hundreds of QPS | Up to tens of TB | OLTP or OLAP          | Moderate     |
| Cold data      | No write operations, immutable   | Rare, individual requests         | Unlimited        | Various storage types | Cheap        |


#### Comparison of Solutions for Warm Data Storage

| **Solution**                                       | **Implementation**                                                                                                                                                                                  | **Pros**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | **Cons**                                                                                                                                                                                                                                                                                                                                                                                                        |
|----------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Partitioning in Existing OLTP Hot Data Storage** | Organize data in tables with partitions and use UUIDv7/UUIDv8 primary keys                                                                                                                          | - reuses the same storage, balancing cost and resources needed for warm storage.<br> - enables atomicity and consistency checks directly on primary storage                                                                                                                                                                                                                                                                                                                                                                                                                                                          | - storage costs increase as warm / hot data grows, which can be expensive.<br> - can slow down the service if queries without partition pruning are run.<br> - requires expertise in primary-key optimizations and partitioning, which may sometimes impact business requirements                                                                                                                               |
| **Offloading Data into a Separate Service**        | Send state event updates for entities with accumulated historical data to a Message Broker (e.g., Apache Kafka) and store history in a large partitioned table, similar to the DealHistory service. | - allows a lower service requirement for the history service compared to the original service due to different access patterns.<br> - endpoints for retrieving historical data should be implemented separately, reducing synchronous call chains and enhancing stability.<br> - supports high-volume history requests.<br> - data quality processes ensure that historical events are fully delivered (at-least-once guarantee) and unique (using primary / unique keys); this stream can also be shared with DWH.<br> - supports integration with the User Termination process and PPI storage in historical data. | - requires additional development effort.<br> - possible delays in data display in the UI due to event transmission through the Message Broker.<br> - Creates a large, inflexible table structure, though this can be mitigated with a JSON field                                                                                                                                                               |
| **Offloading to OLAP Warm Data Storage**           | Use CDC or export old partitions to S3, then load records from S3 into OLAP storage                                                                                                                 | - reduces development resource needs.<br> - allows flexible changes to data access patterns                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | - cannot handle load spikes above tens of RPS consistently.<br> - each team must independently monitor and maintain their OLAP storage.<br> - OLAP storage's low throughput and stability requirements may necessitate actions to maintain service stability according to its [service tiering](service-tiering.md). <br> - low performance of data deletion in OLAP complicates storing PPI in historical data |
