# Architecture Committee Template

| **Item**       | **Details**                                                                  |
|----------------|------------------------------------------------------------------------------|
| Customer       | `@customer_name`                                                             |
| Developer      | `@developer_name`                                                            |
| Repository     | [https://github.com/inDriver/something](https://github.com/inDriver/something) |
| Approved by    | Architects / Backend, Infrastructure, Information Security, DBA,             |
| Project stages | `dd/mm/yyyy` — requirement gathering for the project                         |
|                | `dd/mm/yyyy` — preparation for pre-review at the Architecture Committee      |
|                | `dd/mm/yyyy` — review at the Architecture Committee                          |
|                | `dd/mm/yyyy` — service development, first stage                              |
|                | `dd/mm/yyyy` — service integration, second stage                             |


## General Requirement

List of key tasks and requirements for the project or service in a brief list format:

* task 1 - description;
* task 2 - description.


## Architecture and Service Dependencies

The architectural decision for the project provides the maximum amount of information to all stakeholders and subsequently becomes part of the service documentation.

The service component diagram is presented using a PlantUML diagram in C4 notation. 

>**NOTE**
>
>The use of images in `.png`, `.jpg` formats, and screenshots from screens/IDE, as well as links to Figma/Miro, is prohibited for diagram insertion.


## Tech Stack

The list of technologies to be used for the service is specified as follows:

* gRPC;
* Go;
* MySQL;
* GraphQL.


## Interaction with Other Services

In this section, the connections between services shown in the diagram are briefly explained, and additional details are provided in the form of PlantUML process diagrams.

Interaction between services is achieved through the invocation of a gRPC service, which works with the service's local database and asynchronously notifies listeners by sending events to Apache Kafka.


## ER-diagram and Database Structure

Provide the data schema—this can be in DDL format or any other visual format.


## Calculated Load and Resources Consumption

In this section, any known information about the required resources is provided. This includes load estimation in RPS, required storage size, necessary amount of RAM, and other relevant details.


## Scaling

Show and explain how the service will scale (both within a single data center and across multiple data centers for distributed services).

Describe the operation in geo-distribution / sharding of data. Illustrate possible paths for data replication / sharding between data centers.


## Security

You need to discuss security with the security team—identify any problematic areas and obtain confirmation of the security of the chosen solution.

Highlight vulnerabilities and potential risks / attack vectors that might be visible to the Information Security (IS) team.


## Personal Data

If the service processes personal data, you must provide:

* a list of personal data (PD) being processed and the methods used to ensure its protection;
* the locations where personal data is stored.

Whenever possible, avoid storing personal data directly within your service. Instead, use specialized components with personal data protection, such as user-service, file-storage, and others.


## Reliability

In accordance with the presented component diagram, show how a fault-tolerant solution with high availability will be achieved, in alignment with the infrastructure team.

Specify how the transition to the desired architecture will be managed. Additionally, illustrate the behavior during failures, component outages, and the impact on other services—what will not work if this service fails?

Key decisions:

* the service must be deployed across multiple data centers within a single region;
* the database should operate in a master-master mode. The service connects to the local database through an internal load balancer, which switches to an active database in another data center in case of a failure. Auto-incrementing keys for entities created by the service are prohibited;
* the service has a public `LoadBalancer` that distributes traffic within the region among multiple active instances of the service across different data centers in the same region.

