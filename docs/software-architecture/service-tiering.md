#  Service Tiering Framework


## Overview

In our infrastructure and architecture, managing a wide array of services across multiple data centers presents operational challenges. It has become clear that not all services require the same level of attention, resources, or guarantees requested by business owners. Some services are mission-critical, while others primarily impact our core processes. To address this variability, we propose implementing a service tiering framework. This framework provides a structured approach for classifying our microservices based on various metrics, ensuring that each service receives the appropriate level of attention and resources.


## Decision


### Key Characteristics

After careful consideration of the following criteria, we have decided to implement a service tiering framework, based (but not limited only) on following key characteristics:

* **amount of possible financial loss**: we recognize that different services have different financial implications if they fail. Services classified as less important should have a minimal financial impact if they experience downtime, while services of so-called "Critical Path" may represent a critical financial risk if they fail;
* **cost of service**: we know that the cost of ensuring reliability and availability varies across services. New services may prioritize cost efficiency, while mission-critical services must prioritize maximum stability and availability, regardless of cost;
* **impact on dependent services or processes**: we understand that service failures may have cascading effects on dependent services or processes. The tiering framework should take into account the potential impact on dependents, ranging from "no impact" for general or new services to "possible stoppage of critical processes";
* **impact on users (external and internal)**: we consider the impact of service failures on both external and internal users. All services should aim to minimize impact, besides this, mission-critical services must also prioritize uninterrupted service for external users;
* **psychology**: we recognize the psychological aspect of approving changes and allocating resources to different services. While new or MVP services may have lighter control and can be relatively easy to change, mission-critical
  services should have tighter control and more approvals to avoid disruption of services due to human-factor;
* **SLA (Service Level Agreement)**: we aim to have different SLA targets for each tier, reflecting the varying levels of reliability and availability required. New or MVP services may operate in a best-effort mode with no
  guarantees, giving us enough freedom for experiments, while mission-critical services must aim for near-perfect uptime;
* **non-financial risks / losses**: we acknowledge the existence of non-financial risks and losses associated with service failures, especially in the context of IPO. Our tiering framework must help to identify such risks for us and investors, especially for mission-critical services, where we have risks that are impossible to avoid due to their critical nature. And apply required BCP / DRP policies to such mission-critical services.

Taking into considerations all these aspects, we have agreed to create a **service tiering framework** to efficiently manage wide range of microservices in our architecture. By classifying services based on criteria such as financial impact, cost, user affect, non-financial risks, desired SLA and others, we can allocate resources effectively, prioritize critical services, and mitigate potential failures. This approach should ensure that each service receives the appropriate level of attention and resources, ultimately improving reliability, risk management, and operational efficiency across our system architecture


### Service Tiering Framework

Each service should be scored based on criteria table below to identify own tiering level and to be aware about possible restrictions for given tier.

| Criteria                                  | Tier-1                                                             | Tier-2                                                            | Tier-3                                               | Tier-4                                                  |
|-------------------------------------------|--------------------------------------------------------------------|-------------------------------------------------------------------|------------------------------------------------------|---------------------------------------------------------|
| SLA                                       | Best-effort mode                                                   | 95% SLA                                                           | 99% SLA                                              | 99.95% SLA                                              |
| Allowed async delay                       | Best-effort mode, up to 1 day of delay                             | Up to 72 minutes / day of delay / idle                            | Up to 1 minute / day of delay / idle                 | Delays / idle are not expected                          |
| Amount of possible financial loss         | $ / no affect                                                      | $$ / delays in processing                                         | $$ - $$$ / degradation of key metrics                | $$$ - $$$$ / significant money loss                     |
| Cost of service                           | <$100 / Maximum economy                                            | $100-$1000 / Economy with focus on stability                      | $1000-$10000 / Compromise between stability and cost | >$1000..$10000 / Maximum stability and availability     |
| Affect on dependent services or processes | No affect                                                          | Restricted or low affect                                          | Significant degradation visible on monitoring        | Stop of critical processes, drop in key metrics         |
| Impact on users (external and internal)   | No impact on external users or restricted impact on internal users | Restricted impact on external users or stop of internal processes | External users experience issues with core processes | Stop of critical processes for external users           |
| Psychology                                | Easy to approve, possible quick MVPs, quick architecture reviews   | Growing, still quick, but aligning with architects                | Trade between features and necessity of features     | Hard to approve, slow changes due to change resistance  |
| Non-financial risks / losses              | No losses / risks                                                  | Low probability / low risks                                       | High probability / high risks of losses              | Impossible to avoid risks / losses as they are too high |


#### Special Tier-0 Level Handling

Tier-0 is an additional level in our service tiering framework that require special handling.

This tier is used exclusively in the following scenarios:

* **non-production services**: services that do not yet exist in the production environment, such as newly created or prototype services that are still under development and testing;
* **abandoned services**: services that have been deprecated and abandoned, or removed from production. These services are no longer actively maintained or supported and may be in the process of being phased out;
* **unassessed services**: services that may have not yet undergone an assessment to determine their appropriate tier placement. These services require further evaluation to align them with the correct tier based on their criticality and requirements.

If a service is identified as belonging to the Tier-0, it is crucial to contact the architectural team for guidance. They will provide further discussion on the implications and necessary steps to transition the service to the appropriate tier or to finalize its decommissioning.


### Rationale

The implementation of a service tiering framework offers several benefits:

* **resource allocation**: by classifying services into tiers, we can allocate resources more efficiently, focusing our;
  efforts and investments where they are most needed.
* **risk management**: the tiering framework allows us to systematically assess and manage risks associated with service failures, ensuring that critical services receive the necessary attention and resources to mitigate potential impacts;
* **SLA matching**: service level agreements provide clear expectations for performance and reliability;
* **operational efficiency**: with clear guidelines for service classification and associated SLAs, teams can operate more efficiently, prioritizing tasks and responses based on the criticality of the service.


### Consequences

While the service tiering framework provides clarity and structure for managing microservices, it also introduces complexity and overhead related to classification and maintenance. Tiering requires ongoing monitoring and adjustments during architecture and infrastructure audits to ensure that services remain appropriately classified as their requirements evolve over time.

However, the Architecture Team acknowledges that the benefits of improved resource visibility, risk management, and operational efficiency outweigh these potential drawbacks associated with the service tiering process. Stakeholders and teams need to understand the rationale behind the tiering system to support effective decision-making and alignment.


### Applicability of Solutions Per Tier

To ensure systematic categorization and effective management of microservices, we have created the following table, mapping various features to their corresponding tiers within our service tiering framework. This table provides guidelines for assigning microservices to appropriate tiers based on their criticality, performance requirements, and cost considerations. It can also be used to optimize resources indirectly (by restricting solutions to applicable tiers only), maintain reliability, and prioritize services effectively.

The table content appears well-structured and accurately represents the criteria across the tiers. However, I’ve made minor clarifications to improve readability and consistency:

| Feature                                                 | Tier-1          | Tier-2                    | Tier-3             | Tier-4                                |
|---------------------------------------------------------|-----------------|---------------------------|--------------------|---------------------------------------|
| **Kubernetes**                                          |                 |                           |                    |                                       |
| Minimum number of pods                                  | 1 pod           | 1 pod                     | 2 pods             | 2 pods                                |
| Configured HPA (Horizontal Pod Autoscaler) for non-consumers | Optional        | Optional                  | Required           | Required                              |
| Pod Disruption Budget (PDB)                             | Optional        | Yes                       | Required           | Required                              |
| Usage of multi-AZs in the K8s cluster                   | No              | Optional                  | Required           | Required                              |
| Topology spread / anti-affinity for pods                | No              | Optional                  | Required           | Required                              |
| **ElastiCache**                                         |                 |                           |                    |                                       |
| ElastiCache Burstable cache.t* series (<20% CPU)        | Yes             | Yes                       | Optional           | Only without load                     |
| ElastiCache provisioned cache.*.large series            | No              | No                        | Optional           | Recommended                           |
| ElastiCache shards                                      | Min: 1, Max: 2  | Max: 4                    | Max: 500           | Min: 2, Max: 500                      |
| ElastiCache shard replicas                              | Min: 0, Max: 1  | Max: 2                    | Max: 5             | Min: 1, Max: 5                        |
| ElastiCache shards / replicas autoscaling for >=cache.*.large | No              | No                        | Recommended        | Recommended                           |
| **RDS Aurora**                                          |                 |                           |                    |                                       |
| Aurora serverless v2 instances                          | 0.5–4 ACU       | 0.5–8 ACU                 | Only without load  | Only without load                     |
| Aurora Provisioned >=db.*.large instances               | No              | Optional                  | Recommended        | Required                              |
| Usage of Aurora RDS Proxy (min 2vCPU)                   | No              | No                        | Optional           | Recommended                           |
| Aurora Replicas                                         | Min: 0, Max: 1  | Min: 0, Max: 1            | Min: 1, Max: 15    | Max: 15                               |
| **Network**                                             |                 |                           |                    |                                       |
| Global multi-regional domain names via Route53          | No              | No                        | Possible           | Possible                              |
| Circuit-breaker protection for external calls           | Recommended     | Recommended               | Required           | Required                              |
| **Processes**                                           |                 |                           |                    |                                       |
| SQL DDL Review                                          | Command review  | Command / TL              | TL + DBA + Arch    | TL + DBA + Arch                       |
| Pager duty on call                                      | No              | Recommended               | Yes                | Yes                                   |
| Build / deploy quality gates                            | Optional        | Optional                  | Required           | Required                              |
| Minimum code reviewers                                  | 0               | 0                         | 1                  | 2                                     |
| Deployment restrictions                                 | No restrictions | No restrictions           | 1 region at a time | Canary deployment, 1 region at a time |
| Technology restrictions                                 | >= Assess       | >= Trial, Assess under FT | >= Trial           | = Adopt, trial under feature toggle   |
| **Quality Gates**                                       |                 |                           |                    |                                       |
| Static linters                                          | Required        | Required                  | Required           | Required                              |
| Green unit tests                                        | Required        | Required                  | Required           | Required                              |
| Component API tests                                     | Required        | Required                  | Required           | Required                              |
| Integration end-to-end API tests                        | Optional        | Required                  | Required           | Required                              |
| Load tests                                              | Optional        | Optional                  | Required           | Required                              |
| Security checks                                         | Recommended     | Recommended               | Recommended        | Recommended                           |