# Data Backup and Recovery Policy


## Overview

In today’s rapidly changing business landscape, disruptions can arise from various sources, including natural disasters, cyberattacks, equipment failures, and even pandemics. Ensuring the continuity of inDrive’s business operations despite such disruptions is essential for safeguarding our reputation, maintaining customer trust, and sustaining revenue streams. Therefore, it is imperative to establish a comprehensive Backup and Recovery Policy as an integral part of the Business Continuity Plan (BCP) to mitigate risks and minimize the impact of disruptions through effective Data Recovery.


## Decision

Our company operates a wide range of services, each with varying levels of importance and criticality. These services are categorized into different tiers based on their significance, with Tier-1 representing non-critical services and Tier-4 representing the most critical. Implementing a robust backup and recovery policy is essential to prevent data loss during incidents or disasters and to ensure business continuity. This policy must align with our service tiering model to address the distinct recovery requirements of each service tier effectively.


## Applicability

This backup and recovery policy, along with the data deletion protection mechanisms, applies to all business-critical and non-critical services within our company. The policy is structured to align with the different service tiers, ensuring that each service receives the appropriate level of data protection and recovery based on its criticality and business impact.


## Backup and Recovery Policy Objectives

The primary objective of this backup and recovery policy is to ensure continuous data protection and availability across all service tiers within our organization. The policy seeks to mitigate the risks of data loss—whether from accidental deletions, system failures, or malicious activities—by implementing a robust framework for data backup and recovery.

By aligning the policy with our Service Tiering Framework, we can tailor backup frequencies, retention periods, and recovery objectives (RPO and RTO) to meet the specific needs of each service tier, from non-critical internal tools to mission-critical customer-facing services. Additionally, the policy incorporates stringent data deletion protection mechanisms to safeguard against unauthorized or accidental data deletions.

Ultimately, this policy supports our commitment to maintaining business continuity, enhancing operational resilience, and ensuring compliance with regulatory requirements, thereby securing our data assets and sustaining customer trust:

* **data protection**: ensures that data is not permanently lost due to hardware failures, software bugs, human errors, or malicious attacks. Should provide a safety net, allowing to recover from accidental deletions, corruptions, and other data loss scenarios;
* **business continuity**: maintains operational resilience by ensuring critical services can be restored quickly, minimizing downtime. Should support recovery from major incidents such as natural disasters, ensuring that business operations can resume with minimal disruption.
* **compliance and risk management**: helps meet regulatory and compliance requirements for data protection and retention.
  Reduces the risk of financial losses and reputational damage associated with data loss incidents;
* **service availability**: enhances the reliability of our services by ensuring data is available and recoverable.
  Provides customers with confidence in our ability to safeguard their data.

As part of disaster recovery planning, we have agreed to define an RTO and RPO based on impact analysis and risk assessment.

**RTO** is the maximum acceptable delay between the interruption of an application and the restoration of its service. This objective determines what is considered an acceptable time window for an application to be unavailable. We have agreed that in case of unlucky major outage of one of our critical databases recovery time must not exceed 1h, which has been measured during test recovery procedure of our production databases. Effectively, **RTO=1h** has been approved as a maximum value for all services.

**RPO** is the maximum acceptable gap between the data in the disaster recovery site and the latest data stored in the application when the disaster strikes. This objective determines what is considered the maximum amount of time acceptable for interruption/loss of data that can be caused by a disaster. We do not use advanced recovery features which may increase significantly our costs and rely only on daily backups for all services, thus we MAY have up to 24h of data lost in the unlucky case of disaster. Effectively, **RPO=24h** has been approved as a maximum value for all services.


## Business Impact Analysis and Risk Assessment

To implement properly Backup and Recovery Policy we have decided to map first all our business processes to our microservices and to perform a risk assessment for each service to distribute all our services into several groups, which will have common set of backup and recovery rules and policies, based on criticality of each service and impact on external and internal users. This process is known as "Critical Path" mapping.

As per our [Service Tiering Framework](service-tiering.md), different tiers of services should have varying levels of criticality and importance to our business operations. In alignment with this framework, conducting a periodic detailed business impact analysis and risk assessment must be performed at least once a year to review and check backup and recovery algorithms on production services to ensure that all processes are working and all instructions are up-to-date.

Business impact analysis aims to quantify the impact of an outage on our workloads, taking into account both our internal and external customers. It should assess the impact on our business if workloads become unavailable and helps determine the urgency (or criticality) of their recovery and the acceptable level of data loss.

In alignment with our [Service Tiering Framework](service-tiering.md), this analysis considers the tier classification of our workloads. For Tier-1 services, any disruption may have minimal financial impact and no effect on dependent services or processes. The urgency of restoration and acceptable data loss may be less critical compared to higher tiers. Tier-2 services, while still significant, may allow for slightly longer downtime or data loss, with moderate financial implications and restricted impact on dependent processes. Tier-3 services, being of higher importance, should require swifter restoration efforts and minimal data loss to mitigate significant financial and operational impacts. Finally, Tier-4 services must demand immediate restoration with near-zero data loss, as any disruption could lead to substantial financial loss and halt critical processes affecting both internal and external users.

Therefore, by aligning business impact analysis with our tiered service model, we must prioritize recovery efforts based on the criticality of each workload, ensuring optimal resource allocation and minimizing overall business impact during outages. It's imperative to consider the probability of disruptions and the associated recovery costs alongside recovery objectives. This holistic approach ensures that recovery efforts are aligned with the business value of each workload.

Our risk assessment evaluates the likelihood and potential impact of various disaster scenarios, taking into account the technical and architecture implementation of our workloads. This assessment informs the probability of disruption for different types of disasters. For some cases, it may be justifiable not to have a disaster recovery strategy in place, based on a low probability of certain disaster scenarios. However, it's crucial to remember that we should follow general best-practices to mitigate common disasters impacting only one single instance of our service in one availability zone.

Following part of this document describes in detailed way existing threats, risks, impact, and cost of different disaster scenarios, as well as the associated recovery options. This information must serve as the foundation for determining recovery objectives tailored to the criticality of each workload within our organization.


## Data Loss Prevention

Data loss prevention (DLP) is the discipline of shielding sensitive data from theft, loss and misuse by using cybersecurity strategies, processes and technologies. Data loss prevention (DLP) mechanisms helps inDrive to reduce risks regarding data leaks and losses by enforcing relevant Controls and Security policies on that data to ensure that only the right people can access the right data for the right reasons.

To ensure comprehensive data protection, we have implemented Data Loss Prevention mechanisms tailored to the specific characteristics and requirements of different data storage solutions within our infrastructure. This includes AWS Aurora databases, stateful applications running in Kubernetes, and Amazon S3 storage.


## General Data Retention Policy

Proper data retention is crucial for regulatory compliance, security, cost management, and operational efficiency. Storing data indefinitely increases storage costs, risks of data breaches, and non-compliance with privacy regulations. Conversely, retaining data for too short a period can hinder incident investigation, system monitoring, and business continuity.

We decided to apply balanced Data Retention Policy that defines specific retention periods for different data types. Therefore, all services, components and systems must follow specified Data Retention Periods. This ensures compliance with legal requirements, minimizes security risks, manages costs, and supports efficient access to relevant data for operational and investigative purposes.

| Data Type                | Minimum Retention Period                   | Data Purpose                                                                |
|--------------------------|--------------------------------------------|-----------------------------------------------------------------------------|
| Clairvoyance Telemetry   | 7 days                                     | Monitoring mobile apps health, investigating IT incidents                   |
| Service logs             | 14..30 days                                | Monitoring services health, investigating IT incidents                      |
| Database backups         | 14..30 days                                | Infrastructure data copy to restore in case of corruption or loss           |
| Audit Logs               | 12 months                                  | Tracking user actions in infrastructure and IT systems with SIEM            |
| Access Logs              | 1..3 years                                 | Access logs collected from Cloudfront and stored in S3 with compression     |
| Metrics                  | 2 years (with automated down-sampling)     | Monitoring infrastructure and services, creating alerts in case of problems |
| Cold Database Partitions | Unlimited, according to legal requirements | Holds Offloaded Cold Data Partitions. E.g. up to 7 years for financial data |

The policy balances the need to retain critical data for monitoring, security, and recovery while avoiding the pitfalls of indefinite data storage. Automated data deletion will be implemented to securely remove data that exceeds its retention period, reducing risks and optimizing storage.


## Aurora Data Deletion Protection

AWS Aurora databases are critical components of our data infrastructure, storing essential application data that must be protected against accidental or malicious deletion. To achieve this, [Aurora Delete Protection](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/USER_DeleteCluster.html#USER_DeletionProtection) must be enabled for all AWS Aurora databases. This setting prevents the deletion of Aurora database clusters unless delete protection is explicitly disabled, thereby safeguarding our databases against accidental or unintended deletions and prevents downtimes during recovery procedures.

Deletion of data from databases should be generally avoided, services must be designed in a way that data is preserved inside databases and either never deleted or Cold Data offloaded only when it is not directly needed by such services.

To ensure data protection and high availability for Tier-3/Tier-4 services, we have decided to leverage Amazon Aurora’s built-in multi-AZ replication. Aurora automatically replicates data across multiple Availability Zones (AZs) which increases the availability of data and prevents from unexpected incidents in single AZ. For services in Tier-3 and above, we MUST configure Aurora with at least one read replica in a different AZ from the primary instance. This setup ensures that even in case of an AZ failure, a replica is available for automatic failover, minimizing downtime and maintaining data integrity.

Aurora’s continuous backups to Amazon S3 (see below the Aurora Data Backup Policy), combined with cross-AZ replication, enhance our data durability and recovery in the event of failure. Although this replication increases costs, it provides essential fault tolerance and availability for critical workloads.

For Tier-4 services, additional replicas may be added for further scalability and resilience to cover 3 Availability Zones (AZs) in one single region. This ensures that our critical workloads remain operational even in the event of partial regional outages, aligning with our commitment to high availability for higher-tier services.


## StatefulSets in Kubernetes

For stateful applications such as Apache Kafka, NATS, and ClickHouse running inside Kubernetes, [Kubernetes finalizers](https://kubernetes.io/docs/concepts/overview/working-with-objects/finalizers/) must be added to protect the associated Persistent Volume Claims (PVCs) and StatefulSets to prevent data loss. Additionally, Kubernetes Role-Based Access Control (RBAC) must be used to restrict delete permissions.

So, if someone attempt to delete an object that has a finalizer on it, it will remain in finalization until the controller has removed the finalizer keys or the finalizers are removed using Kubectl. Once that finalizer list is empty, the object can actually be reclaimed by Kubernetes and put into a queue to be deleted from the registry.


## S3 Object Lock Protection

For data stored inside S3 we decided to apply [Amazon S3 Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html), that provides a robust mechanism to protect objects from accidental or malicious deletion. Leveraging S3 Object Lock may enforce write-once-read-many (WORM) policies on S3 objects to meet regulatory requirements and enhance data integrity and can help prevent Amazon S3 objects from being deleted or overwritten for a fixed amount of time or indefinitely.


## Data Backup Policy

Company decided to use [Backup and restore](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html#backup-and-restore) approach for mitigating against data loss or corruption by running automated backup that runs periodically or is continuous where point-in-time recovery is available. While it isn't the most advanced solution, it covers major risks, while keeping cost of Backup/Recovery solutions relatively small.

![](https://docs.aws.amazon.com/images/whitepapers/latest/disaster-recovery-workloads-on-aws/images/backup-restore-architecture.png)

Data in source systems may change at different speed, so we decided that all components may choose custom Data Backup retention period depending on their business needs. However, in any case, the retention period for Data Backups must not exceed 30 days to avoid unlimited growth of backups, recommended Backup Retention time is 14 days.


### Aurora Data Backup Policy

Our company operates distributed services across multiple regions, leveraging AWS infrastructure. We utilize AWS Aurora databases extensively for their high availability and fault tolerance. To prevent data loss during accidents or disasters, robust backup policy MUST be active all the time.

All automated backups of Aurora Databases must be retained for a minimum of 14 days. We decided to implement this rule via the universal AWS Backup policy for our Aurora databases using [AWS Backup](https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html) and Terraform module [`umotif-public/terraform-aws-backup`](https://github.com/umotif-public/terraform-aws-backup) to ensure that all Aurora databases are backed up daily. This policy must be automatically applied to all databases within each region where our applications are deployed.

Below is our current definition of our Backup Rule, see [backup/aurora/plan/terragrunt.hcl](https://github.com/inDriver/terraform-aws/blob/master/templates/backup/aurora/plan/terragrunt.hcl) for actual Backup Policy.

```
    {
      name                     = "${local.name}-rule-${local.aws_region}"
      target_vault_name        = dependency.vault.outputs.backup_vault_id
      enable_continuous_backup = false
      recovery_point_tags      = local.tags

      schedule          = "cron(0 0 ? * * *)"
      start_window      = 60
      completion_window = 180

      lifecycle = {
        cold_storage_after = 0
        delete_after       = 14
      }
    }
```


### Aurora Data Backup Encryption

To safely store all data inside S3, we must store all backups encrypted using AWS KMS (Key Management Service). By utilizing KMS keys, we manage encryption keys centrally and ensure that our backups are protected with industry-standard encryption both in transit and at rest. This also allows for fine-grained access control and audit logging, enhancing the security posture of our backup data.