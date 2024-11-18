# Geo Distribution Principles


## Overview

The company's infrastructure is closely related to the infrastructure of the CSP (Cloud Service Provider). To reduce request latency, distribute load, and increase the resilience of our infrastructure, we strive to build services in a way that they operate in all regions where inDrive is present.

A perfect service should be able to handle requests from any consumer anywhere in the world. However, this presents us with new challenges related to data availability. It's necessary to ensure the ability to read and modify any record in the database from any point on the globe, as well as synchronize these changes across other regions. Since solving this problem with strict consistency guarantees is not feasible, we have introduced standard deployment schemes for services in a distributed infrastructure and the associated terminology.


## Decision


#### Regional Terminology

**Macro-region** — physical entity. A part of the inDrive infrastructure deployed in one CSP region. Macro-regions are mapped to CSP regions in a one-to-one ratio.

**Region** — logical entity. One or more regions can exist within a single macro-region. Typically, a single country generating such a high workload on the infrastructure that it becomes financially justified is placed in a separate region. A full copy of the infrastructure is launched in each region. Regional services can effectively interact with macro-regional services.

**Central Region** — the primary region of the infrastructure where all non-distributed services reside. The AWS region `eu-central1` has been chosen as the central region.


#### Existing Geo-distribution Schemes

**Global undistributed service** — a service launched in one region that handles requests from consumers worldwide. This service has one public address for the entire world.

**Global distributed service** — ensures global availability and data synchronization. From the consumer's perspective, it doesn't matter which specific instance of the service in which region receives the request. This service has one public address for the entire world.

**Macro-regional service** — a service located in macro-regions that processes requests from other macro- and regional services located in the same physical data center of the CSP. Each instance must process its independent copy of data. Each instance of such a service has a separate public address in corresponding macro-region.

**Regional Service** — a service, all instances of which are launched in separate regions. Each instance must process its own part of data independent of other regions. Each instance of such a service has a separate public address in corresponding region.


#### How to Choose the Correct Geo Distribution Schema

When launching a service, the team should primarily consider operating within the AWS infrastructure, as it is the target scheme for inDrive. The decision to launch in FR1 should only be made based on the recommendation of the Architectural Committee. When deciding on the geo-distribution method for a new service, the following set of typical schemes and criteria should be followed:

Consider the regional service scheme as the main one if the following requirements are met:

* each instance has data independent of other regions;
* to distribute high workload;
* if there is a need to exchange events in Kafka with other regional services;
* if data portability between regions is not required;
* cross-regional interactions are absent.

Consider the macro-regional distribution scheme in the following cases:

* if there are no regional dependencies (requests are not sent from the macro-region to regions);
* for cost savings when launching an instance in each region is too expensive;
* low latency request requirements must be met;
* if data portability between macro-regions is not required.

Choose the global service scheme if:

* delays in data replication are unacceptable;
* strong consistency needs to be ensured;
* geo-distribution of such a service requires the organization of a multi-master MySQL cluster;
* there is no requirement for low latency requests;
* the level of fault tolerance accepted for the global service should be sufficiently high. If such a service becomes a single point of failure, its level should be [Tier 3-4](service-tiering.md).

The global distributed service scheme will be preferable if:

* global data availability is required;
* changes to the service data can be eventually consistent;
* the development time and cost of such a scheme are not significantly higher than for the global scheme;
* the blast radius needs to be reduced in case of global service failure;
* if there is a need to reduce the service response time.

These recommendations should serve as a guideline for choosing the service distribution scheme, but they do not cover all possible situation. After selecting the geo-distribution scheme, it must be validated with the Architectural Committee.
