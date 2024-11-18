# Architecture Committee


## Goal

* protection of new projects or the implementation of new solutions before experts to obtain expertise and move forward with confidence that the right decisions have been made;
* increased transparency:
  * the ability to know about the emergence of a new project in advance.
  * the possibility of adjusting the plans of your department (for example, purchasing new servers);
  * staying up-to-date with the latest trends.
* development of agreed-upon quality standards, specifications, and guidelines;
* consideration of regulatory standards' requirements imposed on architecture during implementation, such as GDPR, PCI DSS, and similar.


## Committee

The communication and approval by the committee should be `#architectural-committee` Slack group.

Regular committee members:

* Head of Mobile;
* Head of Backend;
* Head of Infrastructure;
* AppSec Engineer;
* DevOps lead;
* DBA;
* Data Department representative;
* Architects.

Temporary participants:

All interested employees from inDriver are welcome to attend committee meetings. To do so, join `#architectural-committee` and keep an eye on the announcements. No additional permissions are required.

>**NOTE**
> 
> The committee members can opt out of participating in the review (if the project is not related to their team) or send their representatives.
> 
> The absence of a committee member from the project review implies automatic agreement with the decision made.


## Scope of Responsibility

New projects, significant technical decisions, and integration interactions must go through a review process with the committee. The review is conducted by the team's tech lead and / or the project's lead developer.

The review is conducted by the team's tech lead and/or the project's lead developer.


## Architecture Review

* a new project being created from scratch, where the architecture needs to be outlined. Examples: a new microservice is being developed. A new vertical is being created;
* an existing project where significant technical decisions are being changed. Examples: deciding to switch from MySQL to PostgreSQL. Deciding to add Kafka, or to replace direct database calls with the CQRS (Command Query Responsibility Segregation) pattern;
* deciding to use / integrate an existing service that has not been used in the microservice before. Examples: deciding to use a tracking microservice in the vertical to get real-time user coordinates. Deciding to integrate Salesforce.

The architecture review is not required for:

* a temporary project that will exist for a limited time. A project lifespan is defined, which cannot exceed a set duration (3-6 months). After this time, the project must either go through a review or active development should be completed;
* an MVP that will be rewritten later. Similar to a temporary project, an evaluation period is set for the results of the MVP project. If everything goes well, and it smoothly transitions to active use in production, a review is required;
* a small utility for internal tasks that performs a limited function. Example: a utility that uploads files to the cloud;
* product development that fully fits within the existing service architecture.


## Architecture Committee Process

Conceptually, the process of obtaining approval from the Architecture Committee for a service or product under development is outlined below:

* a product or technical idea for implementation emerges;
* the idea is discussed with the architecture team / individual members of the Architecture Committee to form a preliminary implementation plan and a draft of the technical solution;
* an empty epic is created for this project in the Jira Arch Committee project, within which tasks are created to prepare a design document in Confluence based on the [template](architecture-committee-template.md);
* pre-review of the design document with Architecture Committee members is performed after the task for preparing the architecture document is moved to the "Review" status (see the "Pre-review" section below). It is conducted in an informal format—individual or group meetings—during which adjustments are made to the document until consensus is reached;
* only after a successful pre-review is the development of the service allowed to begin. Writing any code before the pre-review is not recommended, as the database and chosen solutions may change;
* after receiving approval of the design document during the pre-review, a full review must be conducted.


### Pre-Review Process

After the design document is prepared, a pre-review of the project with specific committee members is required. A short meeting is held, where the part of the project that is of interest to the committee member is discussed in an informal format.

>**CAUTION**
>
> Without a pre-review by the specified individuals, the project cannot proceed to the full review.

The pre-review should be done with the participation of:

* Architects;
* DBA;
* Infrastructure Department;
* Product Security Department.


### Review Process

* the project representative notifies the committee members in `#architectural-committee` that a project review is needed, sharing the design document for their reference. A time for the review is scheduled by selecting an available slot (considering the current workload of the members—typically about a week). It's recommended to schedule the review meeting in advance, right after the project's pre-review.
* provide the link to the documentation for review by the committee in `#architectural-committee`;
* at the review meeting:
  * the representatives present the project, with the presentation not exceeding 15 minutes;
  * this is followed by a discussion. Experts ask questions, and the representatives provide answers. The discussion lasts around 30 minutes;
  * the meeting is being recorded;
* after the discussion, a decision is made regarding the project. The decision can be one of the following:
  * approve the project (possibly with comments);
  * reject the project, providing reasons;
  * send it back for revision, providing reasons. The project representatives may also withdraw the project for revision based on the discussion results.


### After the Review

* following a successful review and addressing any comments, the design project becomes the basis for the project's technical specification;
* it becomes part of the service's documentation and should be located in a repository;
* the meeting discussion result are posted in the service documentation in a repository. 