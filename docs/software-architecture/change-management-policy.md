# Change Management Policy


## Context

The organization recognizes the need for a formal change management policy and process to ensure that all changes are correctly assessed, managed, and include adequate and proportionate consideration for information security and privacy. This document is part of our Business Continuity Plan (BCP) and Disaster Recovery Plan (DRP), and aligns with the PCI-DSS framework, GDPR framework, and all other relevant regulatory documents applicable to our business.


## Decision

inDrive has decided to adopt and implement a formal change management policy and process to ensure that all changes are thoroughly assessed, managed, and incorporate adequate and proportionate consideration for information security and privacy aspects.


### Applicability

This policy applies to all infrastructure, technology, and personnel involved in inDrive’s change management process, encompassing all changes to infrastructure. This includes, but is not limited to, modifications to in-scope networks, system component configurations and updates, software configurations and updates, deployment of new system components, deployment of new software, and other relevant changes. The documented change management process MUST be followed for initiating, recording, reviewing, approving, and executing all changes within the in-scope environment. Any changes made outside of the change management process are strictly prohibited.


### Distribution, Reviews, and Updates

Software Architect is responsible for making this document available in the form of ADR stored and ensuring that all relevant personnel are familiar with its content and have acknowledged the assigned responsibilities.

Cybersecurity Compliance Manager is responsible for reviewing the current document at least once a year in order to ensure the document’s relevance to the inDrive’s business and risk environment, and its compliance obligations.

Software Architect is responsible for updating the current document upon every change to the entity's environment, in order to maintain its relevance to the inDrive’s business and risk environment, and to ensure the ongoing compliance.


### Roles and Responsibilities

All responsibilities for the management, documentation, implementation, and maintenance of inDrive’s Change Management Policy, procedures, and related activities are formally assigned to the following users, teams, and roles:

| Team / Contact Person                                                                                                  | Role                             | Responsibility                                                                                                  |
|------------------------------------------------------------------------------------------------------------------------|----------------------------------|-----------------------------------------------------------------------------------------------------------------|
| Software Architect team      | Software Architect               | - overall responsibility and accountability for the change-management processes;<br> - policy review and update |
| Cybersecurity Compliance | Cybersecurity Compliance Manager | - Policy approval                                                                                               |
| inDrive employees                                                                                                      |                                  | - Implementation of the Policy                                                                                  |


### Change Management Objectives

The purpose of these objectives is to ensure that changes to the organization, business processes, information processing facilities, and systems that affect business continuity, information security and privacy are adequately controlled.

inDrive Change Management policy provides a structured approach to managing all system changes. Key objectives of the policy covers:

* **formalized process**: the formal change control process should be defined and documented. Changes to information resources including systems, processes or their components, must be managed and executed according to this defined formal change control process;
* **verification**: all change requests should be prioritized in terms of benefits, urgency, the effort required and potential impact on operations;
* **approval**: the control process should ensure that the changes proposed are documented, reviewed, authorized, tested, implemented, and released in a controlled manner;
* **traceability**: the status of each proposed change should be tracked via relevant ticket or linked change request;
* **compliance**: change control process must be in place to control changes to all critical company information resources (such as hardware, software, system documentation and operating procedures). Changes to business-critical software packages should be discouraged unless necessary.
* **segregation of duties**: enforces separation between development, testing, and production environments.


### Types of Changes

Different types of changes may require varying levels of control and management due to their potential impact on systems and services. The change management policy classifies changes into following categories:

* **pre-approved routine tasks**: these are low-risk changes that follow a predefined procedure and have minimal impact on systems, components and all applicable entities (see applicability). They are pre-approved due to their routine nature and can be executed without further authorization;
* **regular tasks**: these changes are not routine but also not critical. They require a standard approval process, including documentation, impact analysis, and testing, before implementation;
* **critical change tasks**: these are high-risk changes that could significantly impact critical services, systems, components (including mobile packages), infrastructure and all applicable critical entities (see applicability). They require careful assessment, approval by reviewers and comprehensive testing to ensure stability and security.


### Pre-Approved Routine Tasks

To streamline operations and enhance efficiency, certain routine tasks performed on a day-to-day basis may be pre-approved and, therefore, may not require adherence to standard change management procedures.

These tasks include, but are not limited to:

* daily system backups;
* routine software updates;
* regular security patch installations;
* standard user account maintenance activities such as password resets and account unlocks.

These pre-approved tasks should be documented and reviewed periodically to ensure they remain low-risk and do not impact the overall stability and security of the IT environment. By pre-approving these routine activities, we aim to reduce administrative overhead and enable IT staff to focus on more critical and strategic initiatives, while still maintaining a controlled and secure operational environment.


### Critical Change Tasks

When the change to the in-scope environment impacts critical processes, for example, services (described as Tier-3 / Tier-4 in the Tiered Service Model), systems, components (including mobile packages), networks, relevant cloud infrastructure or any applicable item (see applicability), it should be considered as a significant change. Whether a change is significant or not is determined via architecture, security or business impact analysis which is part of the change management process.

Each of the following activities, at a minimum, has potential impacts on our critical elements and must be considered as a significant change in the context of related BCP / DRP requirements:

* new hardware, software, cloud or networking equipment added or changed in the context of such critical items;
* any replacement or major upgrades of hardware or software;
* any changes in the business critical processes residing in any parts, responsible for such processes;
* any changes to the underlying supporting infrastructure, responsible for such critical processes (including, but not limited to network services, gateways, vaults or key stores, logging and monitoring).


## Change Management Policy

### Pre-Production Environments

inDrive maintains pre-production environments that are separated logically from the production environment. Beside logical separation of environments, access controls are implemented via RBAC and user roles to enforce further separation of duties between users and environments in order to:

* minimize the number of personnel with access to the production environment, critical services and their components;
* minimize the risk of unauthorized, unintentional, or inappropriate access to data and system components;
* minimize the risk of fraud or theft since two employees would need to act together in order to hide their activities.

Roles and functions should be also separated between production and pre-production environments to provide accountability such that only reviewed and approved changes are deployed. All test data and accounts must be removed from all system components, files, databases, etc. prior to the systems’ deployment into production.


### Change Management Process

inDrive has established, maintains, and follows a Change Management Procedure for all changes to the in-scope environment.

To track all changes, company uses dedicated systems to store and track changes on Enterprise level for all in-scope systems and components, like [Git](https://git-scm.com/) - distributed version control system that tracks versions of files and keeps history of all changes. All changes should be done in the form of Change Request inside isolated repositories.

The following steps outline the procedure, which will be described further in details:

* **change request submission**: changes may be initiated by any stakeholder through a standardized change request form in Jira. Each change ticket (or task) MUST include the following elements as a minimum:
  * identifier of project where change SHOULD be introduced;
  * reason for the change;
  * description of the change, which may also include documentation of business, security or architecture impact;
  * priority of change (Critical / High / Normal / Low).
* **assessment and approval**: each change request should undergo an impact analysis to assess potential risks to business, security, privacy, architecture and system stability. Once reviewed, requested changes may be approved, adjusted or rejected by relevant stakeholders and system owners. During assessment phase additional check-Lists may be added to a ticket in the form of Definition-of-Ready (DoR) and Definition-of-Done (DoD). Teams also may use additional fields in their projects to specify meaningful details like Start Date, Planned Completion Date, Scheduled Sprint, Performer, etc.;
* **implementation phase**: each regular change request should be represented in the form of atomic code change where applicable (either Git commit or Git branch with series of commits in it) or by performing actual changes inside systems or components. Such changes typically represented as Git commits or Git branch name must contain unique identifier of ticket from Jira to be able to link and to trace such changes to the relevant approved and documented change request. For each change following information must be available:
  * reason for change (unique identifier of change request, usually Jira Ticket number);
  * date of change — when changes have been introduced;
  * change itself — what changes have been introduced;
  * performer — unique identifier of user, who was introduced these changes.
  
> **WARNING**
>
> If state of systems and components can not be represented as a code inside company repositories, changes for such systems or components may be tracked just inside relevant ticket in a form of comments, describing and confirming the change. For such changes there should be clear evidence that performer has completed the requested change properly and moved the task to the "Done" status.

* **review phase**: each change requested should be reviewed before accepting (merging) proposed changes. Different system, components and services may have different requirements and policies regarding the necessary things to check. This ensures that the change aligns with regulatory requirements, internal policies and all necessary documentation is complete and accurate. Additional policies, Automated Tests or Quality Gates may be used to verify proposed changes and restrict unverified, untested or broken changes from being deployed to the production environment;
* **change testing**: each change (where applicable) should be covered with series of automated test and tiering requirements. All testing should be conducted in dedicated isolated development or testing environment that mirrors (or reproduces as close as possible) the production environment. Testing on production environment must be generally avoided to prevent untested changes on production environment. Testing results should confirm that the proposed change does not adversely impact system functionality, stability or security and new changes are working as expected according to the ticket description;
* **change authorization**: once change request is reviewed, tested, verified, approved and ready — it may be authorized for deployment to production, for release or for actual performing the change. This requires necessary role (or explicit RBAC permissions) to authorize change for the production, for release such changes or for performing the actual change. For system, components and all entities represented as a code (where applicable) this must involve merging proposed changes into the main branch of the corresponding service, component or system. Some important parts of the code (like core functionality, migrations, algorithms) may be additionally protected by specifying the code-owners for such important parts, read more at [GitHub - About Code Owners](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners). This ensures proper authorization of changes and explicit approval from responsible persons according existing policies and procedures;
* **release procedure**: at this step, responsible person, who has necessary privilege (via RBAC) to deploy to production, to release a new version or to make actual change, performs deployment of change to the production, release of a new version, or making actual change. In case of introducing changes to all regions, such changes must be restricted via canary releases and artificial timeouts between introducing changes to all regions to reduce the chances of broken changes to all regions and systems. Responsible person should continuously conduct a review of metrics and dashboards after applying the change to ensure that the change meets its objectives and does not introduce any new issues. Responsible person should be prepared to address any incidents or issues arising from the change.