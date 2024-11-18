# Universal Distribution of City and Country Settings


## Overview

In the inDrive, we have historically stored city and country settings in separate MySQL tables in the monolith database (`tbl_city` and `tbl_country` respectively) with extensive amount of columns, including a JSON field `jdoc` to accommodate changing configuration needs.

This approach served us well in the era of monolithic applications but became challenging when we transitioned to a microservices architecture. The main challenges were related to:

* configuration complexity: managing city and country settings across multiple microservices, distributed across different regions, became increasingly complex and error-prone;
* synchronization: we faced the need to synchronize these settings efficiently and reliably between services and regions, considering network errors and service failures.


## Decision


### Distributed City Settings Model

To address the challenges outlined above, we have decided to implement a universal scheme for distributing city and country settings. This scheme involves the following key components:

* dedicated MySQL database: dedicated MySQL database must be prepared that will exclusively store all city and country settings. This database should serve as the centralized source of truth for city and country configurations;
* regional replication: this settings database must be replicated to all our main regions. Each region must maintain an individual regional replica to survive main region failures. One of these regions will act as the primary / master instance responsible for controlling settings worldwide;
* to streamline configuration updates and minimize constant data fetching, we have implemented the `geo-config` service. This service acts as a logical replica client and subscribes to the regional database's binlogs, allowing service to perform near-realtime replication of changes to the stored state;
* `geo-config` client package: any Golang services that want city and country settings should be discouraged from maintaining their own city and country tables. Instead, they should utilize the `geo-config` client package. This package facilitates seamless communication with the nearest regional instance of the `geo-config` service, ensuring that services receive timely updates to their configurations.


### Configuration Hierarchy

Configuration of dozen cities in one country can be very time-consuming and error-prone process, this is why we decided to enable inheritance of settings for cities from parent items, such as `agglomeration` and `world`. Pay an attention, that `region` has be renamed to the `agglomeration` to avoid collision with the AWS regions, which are actively used.

So, here is our hierarchy of settings, which covers most of our needs:

```
World > Country > Agglomeration > City
```

All settings from low-level records are merged with settings from the parent items, eg. if there is a country setting, then all cities inside this country will inherit this setting too (but can override value if needed).


### Consequences

This architectural decision brings several benefits:

* simplified configuration management: services must not store their own city / country configuration tables, therefore reducing complexity of each such service and avoiding the need of constant DDL changes for such tables;
* global consistency: the primary region must serve as a control point, ensuring global consistency in city and country settings across all the world;
* improved fault tolerance: regional replication should enhance the fault tolerance by providing configuration replicas across different macro-regions;
* near-realtime updates: services should receive near-realtime updates to configurations, improving responsiveness and minimizing latency.

However, there are also considerations:

* dependency on the `geo-config` service: services become dependent on the `geo-config` service for configuration updates and in case of failures, they may have issues during initialization phase or in runtime. In such cases, application should either start in graceful mode or fail the K8s startup probe to avoid disruption of service;
* database maintenance: separate regional MySQL database requires regular maintenance and monitoring to ensure reliability. Replication lag should be proactively monitored to avoid situations when we can temporary lost the control over lagged region;
* setup complexity: implementing this architecture requires initial setup and configuration efforts. However, once configured this scheme keeps working without extra maintenance.