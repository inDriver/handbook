# Pre-delivery Process Shares Services

Two meetings per week, each lasting 60 minutes, will be held to cover all three stages of the pre-delivery process. The agenda for the meetings includes:

* refining the Product Description;
* technical Decomposition;
* estimation and DoR Compliance Check.

The stages are sequential and not tied to specific meetings. During a single meeting, we might quickly discuss the product description and move on to technical decomposition.


## Refining the Product Description

**Responsible:** Product Manager .


### Pre-requisites

**Product Manager:**

* create a User Story in Jira;
* prepare the product description in Jira: implementation goal, user problem being solved, acceptance criteria, etc;
* create a process diagram (flowchart) for the specific story;
* ensure updated UI mockups are available if UI implementation is required;
* one day before the meeting, share the stories / features to be discussed in the Slack channel, tagging necessary team members for task discussion.

**Development Team:**

* review the proposed stories on time;
* ask clarifying questions about the product description in the thread or reserve them for the meeting.


### Actions During the Stage

* discuss the product description;
* ask questions about goals, acceptance criteria, and the user problem being solved.


### Outcome

* answered questions regarding the product description;
* updated User Story (if necessary);
* if questions remain unanswered during the meeting, the PM will take the User Story for further refinement.


## Technical Decomposition

**Responsible:** Engineering Manager.

### Pre-requisites

* refined product description: task goal, description, acceptance criteria;
* updated UI mockups, if required;
* a list of requirements from other teams and deadlines, if dependencies are known in advance.


### Actions During the Stage

* the team discusses what needs to be done to implement the functionality according to the product description, preparing a test plan;
* if technical research is needed, the team creates an enabler, specifying the research goal and acceptance criteria, assigns it to an assignee, sets a completion deadline, and estimates it in Story Points;
* the assignee briefly reports the research highlights in the team’s Slack channel. If necessary, a meeting is convened to discuss the research. If no questions arise, the assignee continues the research;
* the possibility of decomposition is explored;
* dependencies on other teams are identified and resolved.


### Outcome

* a technical breakdown has been conducted: general functional scheme, contracts;
* internal / external dependencies are identified, with those threatening the completion of the User Story within one sprint resolved;
* the task is technically described and divided into sub-tasks: frontend, backend, which are then transferred to Jira;
* a task for automated tests is created (if applicable).
* test scenarios are described.
* risks have mitigation scenarios.


## Estimation and DoR Compliance Check

**Responsible:** Engineering Manager 

### Pre-requisites

* all task information gathered in the previous stages;
* this information should be compiled in Jira within the task (links, text, etc., everything necessary).


### Actions During the Stage

* estimates are assigned to the investigated tasks if the decomposition is complete;
* tasks are checked for DoR compliance (If a task does not meet DoR, the issue is identified and sent for revision);
* a decision is made regarding the acceptance of the task into the sprint.


### Outcome

* the task meets DoR;
* an estimate is provided.
