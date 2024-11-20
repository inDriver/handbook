# Jira Workflow for Product and Platform Teams

The current workflow in Jira is managed from two points: IT&S Automation Team and PMO Team

Each workflow is customized for a specific cluster (cluster-based approach), meaning teams within the same cluster use standardized statuses. However, they can request validators for certain transitions (e.g., mandatory field checks) without impacting workflows in other clusters. To modify your workflow or add something to standard pool of statuses, please reach out to a PMO representative.


## Issue Types


### Epic

The **Epic** issue type in Jira represents a significant feature or service enhancement that delivers business value and addresses customer needs. It is broken down into smaller **Stories** and further divided into **Subtasks**, allowing teams to structure work hierarchically, clarify goals, and make progress on larger initiatives incrementally. Epics often involve multiple teams and may span across various projects and boards. They are typically tracked using parent/child relationships or an **Epic Link**. Unlike individual stories, Epics generally do not require estimation and may extend over multiple sprints.

**Label** is used as a field type in Jira for categorizing and filtering issues. Labels are case-sensitive, can be generated as needed, and are not unique to any single project, making them versatile for filtering across projects.

The **Epic** type is used when a feature can be decomposed into multiple Stories, and tracking progress through logical connections is necessary. A global Epic is created when there is a broader business need or initiative encompassing multiple related features.

After **OKR planning**, Epics are created based on the maturity of the Key Results (KRs). These Epics can represent the Key Results themselves or serve as initiatives under the KRs.

An Epic outlines the business goal and the high-level roadmap for implementation. Child issues, such as Stories, Enablers, and Tasks, are organized under the Epic. To group multiple Epics within a single global initiative, the **label system** is used, enabling an Epic to be part of multiple initiatives.


### Story and Enabler

**Story** — is the most common type of work item in team, describing a small piece of functionality that the Team can complete within one sprint. While most stories are created by breaking down a larger feature, some stories are independent and arise from the team's understanding of its work.
  
Despite the origin, all stories are sent to the team’s backlog. Any team member can write a story, but only the **Product Manager** determines what goes into the backlog and what does not. Product Manager also evaluates the result when work on the stories is completed.

An **Enabler** represents work related to architecture, compliance, research, or infrastructure that is necessary to support ongoing development. Because this work does not directly impact users, it is typically described using more technical language.

* each Story / Enabler should represent a small, manageable piece of work that the team can complete within a single sprint, supporting incremental development;
* Stories / Enablers must have clearly defined acceptance criteria that provide all necessary information for the team to implement the story correctly;
* the development of a story/enabler in most cases should be finished within a 2-week period.


#### Sub-Task

A **Sub-task** is used to break down stories into more detailed components. Sub-tasks serve as a to-do list helping recall what needs to be done in a particular Story, even after some time has passed.

The following types of subtasks exist in our Jira:

* **Backend Task**;
* **Android Task**;
* **iOS Task**;
* **Analytics Task**;
* **Design-task**;
* **Sub-task** is used in different cases and by different roles. For example, a QA Engineer might use a sub-task to write test cases for a specific story.


### Bug

A **Bug** is an issue type that represents a problem or defect in the software or product being developed.


### Task

A **Task**  is often used for non-development activities, such as actions identified by the team during a retrospective.
