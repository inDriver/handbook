## Pre-delivery Process Shares Services

Two meetings per week, each lasting 60 minutes, will be held to cover all three stages of the pre-delivery process. The agenda for the meetings includes:

- Refining the Product Description
- Technical Decomposition
- Estimation and DoR Compliance Check

The stages are sequential and not tied to specific meetings. During a single meeting, we might quickly discuss the product description and move on to technical decomposition.

---

## Refining the Product Description

**Responsible:** Product Manager 

### Pre-requisites:
**Product Manager:**

- Create a User Story in Jira.
- Prepare the product description in Jira: implementation goal, user problem being solved, acceptance criteria, etc.
- Create a process diagram (flowchart) for the specific story.
- Ensure updated UI mockups are available if UI implementation is required.
- One day before the meeting, share the stories/features to be discussed in the Slack channel, tagging necessary team members for task discussion.

**Development Team:**

- Review the proposed stories on time.
- Ask clarifying questions about the product description in the thread or reserve them for the meeting.

### Actions During the Stage:
- Discuss the product description.
- Ask questions about goals, acceptance criteria, and the user problem being solved.

### Outcome:
- Answered questions regarding the product description.
- Updated User Story (if necessary).
- If questions remain unanswered during the meeting, the PM will take the User Story for further refinement.

---

## Technical Decomposition

**Responsible:** Engineering Manager 
### Pre-requisites:
- Refined product description: task goal, description, acceptance criteria.
- Updated UI mockups, if required.
- A list of requirements from other teams and deadlines, if dependencies are known in advance.

### Actions During the Stage:
- The team discusses what needs to be done to implement the functionality according to the product description, preparing a test plan.
- If technical research is needed, the team creates an enabler, specifying the research goal and acceptance criteria, assigns it to an assignee, sets a completion deadline, and estimates it in Story Points.
- The assignee briefly reports the research highlights in the team’s Slack channel. If necessary, a meeting is convened to discuss the research. If no questions arise, the assignee continues the research.
- The possibility of decomposition is explored.
- Dependencies on other teams are identified and resolved.

### Outcome:
- A technical breakdown has been conducted: general functional scheme, contracts.
- Internal/external dependencies are identified, with those threatening the completion of the User Story within one sprint resolved.
- The task is technically described and divided into sub-tasks: frontend, backend, which are then transferred to Jira.
- A task for automated tests is created (if applicable).
- Test scenarios are described.
- Risks have mitigation scenarios.

---

## Estimation and DoR Compliance Check

**Responsible:** Engineering Manager 

### Pre-requisites:
- All task information gathered in the previous stages.
- This information should be compiled in Jira within the task (links, text, etc., everything necessary).

### Actions During the Stage:
- Estimates are assigned to the investigated tasks if the decomposition is complete.
- Tasks are checked for DoR compliance (If a task does not meet DoR, the issue is identified and sent for revision).
- A decision is made regarding the acceptance of the task into the sprint.

### Outcome:
- The task meets DoR.
- An estimate is provided.
