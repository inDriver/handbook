# Data General Protection


## Overview

As data security becomes increasingly critical in today’s digital landscape, our company is committed to implementing robust measures to protect the integrity, confidentiality, and availability of all data assets.

This document outlines the general principles and methodologies adopted by inDrive (hereinafter referred to as "the Company") to safeguard sensitive information from unauthorized access, breaches, and other potential security threats.

In the context of this document, sensitive data refers to information of special commercial value to inDrive, such as trade secrets. Access to such information is restricted based on job functions or duties and is granted only to a limited range of recipients.

The main objectives of the document:

* ensuring data protection in the Company;
* creating a unified data protection approach in the Company.

The approaches outlined in this document are designed to provide a comprehensive framework for data protection, ensuring compliance with industry standards and legal requirements.

This document was developed in accordance with the requirements of PCI DSS, GDPR, ISO/IEC 27001:2022, ISO/IEC 27701:2019, and other relevant legal requirements and standards.


## Decision

All responsibilities for the management, documentation, implementation, and maintenance of Data Protection policies, procedures, and related activities are formally assigned to the following users, teams, and roles:

| Team / Contact Person                                                                                                          | Role                                   | Responsibility                                                                                                                                                                                                                          |
|--------------------------------------------------------------------------------------------------------------------------------|----------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Software Architect team                 | Software Architect                     | - Accountable to enforce data protection policies in infrastructure and services                                                                                                                                                        |
| Cybersecurity Compliance         | Cybersecurity Compliance Manager       | - Responsible to define applicable cybersecurity legal requirements and standards <br /> - Responsible to run and continuously improve security assessment processes <br /> - Responsible to review and update data protection policies |
| Privacy&Product Compliance | Director of Privacy&Product Compliance | - Responsible to define applicable privacy legal requirements and standards <br /> - Responsible to run and continuously improve privacy assessment processes <br /> - Responsible to review and update data privacy policies           |
| inDrive employees                                                                                                              |                                        | - Implementation                                                                                                                                                                                                                        |


## Data Encryption & Masking

Data encryption is a fundamental security measure, essential for protecting sensitive information within our systems. We require to ensure the data confidentiality and integrity, both at rest and in transit, by applying measures of different nature, including but not limited to the use of encryption. Specifically, encryption must be applied to Relational Database Service (RDS) storages, Elastic Compute Cloud (EC2) and all physical volumes by using drives with Self-Encryption (SED) Capability, safeguarding data stored on disks and other media from unauthorized access.

> **NOTE** 
> 
> There are some old nodes in our bare-metal FR01 cluster (servers.com) running on drives that are not SED compatible. These nodes are subject for update in 2025.

To maintain the security and privacy of data processed by the Company, production data must not be used in test environments without proper masking. If masking is impractical or too challenging to implement, synthetic data must be used instead. Additionally, sensitive data should be masked in external communications to prevent unauthorized access where applicable. Similarly, sensitive information must be masked in logs to ensure that confidential data is not inadvertently exposed through logging mechanisms.

Documents that describe how the Data Encryption & Masking is implemented in the Company can be found in the list below.


## Access Control

Implementing access control policies is essential to ensure that only authorized individuals and services can access sensitive data within information systems. These policies help protect data integrity, confidentiality, and availability by restricting access based on roles, responsibilities, and the principle of least privilege.

The Сompany's approach to access management is based on principles such as:

* deny-by-default;
* need-to-know;
* least privilege;
* role-based Access Control (RBAC);
* information classification and the value of information assets;
* use of encryption;
* privacy by design;
* privacy by default.

This means that authentication and Role Based Access Control (RBAC) policies must be applied for Databases, NoSQL storages, Data Warehouses (DWH), Backups, S3 storages, Kubernetes Clusters, server infrastructure and administrative panels. From the network perspective two fully separate contours must be utilized: dev and prod to minimize the risk of unauthorized access and potential data breaches, ensuring that sensitive information is inaccessible outside the production environment.


## Employee Awareness

The Security Awareness program is designed to equip employees with the necessary knowledge and skills to safeguard the company's Sensitive Data and other assets.

This program is facilitated through a series of courses, tests, and communication channels available on the TalentLMS platform.

The company conducts regular phishing exercises to teach employees how to respond correctly when receiving phishing messages. The results of these exercises are analyzed to gather metrics and improve employees' responses to potential attacks.

Regular CTF events are held for the Application Security Team, where employees share their experiences.

Communication Channels in messenger:

* `#security-awareness`: all security-related announcements are posted in this channel;
* `#performance`: this channel is used to reach a wider audience within the company. Any security-related questions can be asked here;
* `#product-security`: a channel dedicated to product security announcements;
* `/helpbot`: a bot where you can ask any security-related questions, and it creates a task.


## Change Management

The Company has established a change management approach to ensure that all changes are properly assessed and managed, with appropriate and proportional consideration given to information security and privacy aspects.

Different types of changes may require varying levels of control and management due to their potential impact on systems and services.

The change management approach classifies changes into following categories:

* pre-approved routine tasks;
* regular tasks;
* critical change tasks.

The change management approach is based on the following phases:

* change request submission;
* assessment and approval;
* implementation;
* review;
* change testing;
* change authorization;
* release procedure.


## Threat Analysis

To enhance inDrive’s data security, regular threat analysis of our infrastructure and services should be conducted. All infrastructure components and external dependencies must be assessed for publicly disclosed vulnerabilities. If vulnerabilities are identified, patches and bug fixes should be applied to the system as soon as they become available.


## Regular Audits and Compliance

The retention periods for sensitive data within the company vary and depend on the type of information being processed.

PI of data subjects is processed in accordance with data retention periods established by legislation and the procedures set by the company.

All logs and backups generated by infrastructure and services may contain sensitive information therefore a retention period is defined for this kind of information before deletion. Depending on type observability data and backups may be stored no longer than specified in the "Observability Standard". This policy ensures that no sensitive information will be stored in logs and backups longer than specified in "Personal Information Handling".


## Regular Audits and Compliance

Regular security audits must be conducted to ensure compliance with relevant data protection regulations.

The company regularly conducts internal and external information security audits.

Internal audits, supported by comprehensive checklists like AWS Infrastructure Checklist and Service Maturity Model, and such international standards as ISO/IEC 27001:2022 and ISO/IEC 27701:2019 are integral to our data protection approach, enabling us to systematically evaluate the security status and address any weaknesses or compliance gaps.

External information security audits are carried out with the assistance of external consultants as well as during information security certification processes.

To obtain an objective assessment of the infrastructure security status, regular penetration tests must be conducted.

The company conducts both external and internal penetration tests, and based on the results, a roadmap is created for addressing identified vulnerabilities in information systems.

In total, this ongoing processes ensure that our Data Protection approach is continuously monitored, policies maintain a robust security framework and data protection practices align with the applicable legal requirements and industry standards.