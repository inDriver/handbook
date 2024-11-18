# Documentation Day Guide

This guide is for Documentation Day, where our primary aim is to dedicate focused time to writing and thoroughly address gaps in our technical documentation.

It is recommended that Documentation Day be held at least once a quarter

**Documentation Day is scheduled as follows:**

* **in-person:** for participants who prefer office work:
  * a meeting room is booked;
  * catering arrangements are made.
* **online:** a common online meeting will be available for participants.

Quick support is also available in the `#documentation-day` and `#documentary` Slack channels.


## Preparation for Documentation Day

Preparation for Documentation Day should begin at least one month before the event.

* identify a documentation issue or gap to be addressed during the event;
* coordinate with the Engineering Managers (EM) and Heads of Engineering (HoE) to agree on the upcoming event and secure the participation of their teams. Communication should be conducted both online and offline in the `#documentation-day` channel. Ensure that managers are added to this channel. The primary goal is to inform the managers and confirm the participation of their team members in the event;
* create a list of participants who will contribute to the documentation;
* find 2-3 individuals who are willing to participate solely as reviewers;
* create Jira tasks in collaboration with the participants. It’s important that participants review and verify the task descriptions, so they are fully informed about their assignments. Assign these tasks to the participants, and ensure they are added to the Jira dashboard using the `documentation-day` tag;
* if a new template is being used for documentation, test it with one or two teams;
* arrange an introductory meeting two weeks before the event. This meeting should include all participants and cover the template, task dashboards, and the workflow for assistants and reviewers during the event. If a task assignee is not present at the introduction, reach out directly to confirm their readiness and the validity of the task;
* after the introductory meeting, book a time slot for the event and set up a Google Meet session;
* share informational posts on documentation writing that will be useful during the event;
* prepare an image outlining the event workflow for participants, which will be shared via posts in Slack channels.
* announce each update about the event in the following channels: `#dev`, `#documentation-day`, and `#documentary`. The final announcements should be made one week and one day before the event;
* arrange catering in advance for those attending the event in person.
* reviewers and the event organizer should support participants via Google Meet and in the `#documentation-day` channel;
* two days after the event, post the results and recognize the best contributors (if applicable), and distribute a feedback survey;
* distribute prizes for contributions to the documentation among the participants;
* organize a retrospective with the organizers and any interested parties;
* address any feedback if necessary.


## Step-by-Step Instruction for Participants for the Event

> **NOTE**
>
> Docs are to be written in the **English language**.

* move the Jira task to the **IN PROGRESS** status;
* create a branch and a PR, including the task number as a prefix in the name. Example: `GEO-2912-branch-name`. If you want to render documentation from a branch on the platform, check the `.documentary.toml` file to ensure the regular expression or branch name is correctly specified in the `branch` field;
* archive and deprecate pages in Confluence or other platforms, if they exist;
* when the PR is ready for review, assign it to the relevant reviewers and to `agoncharova-indriver`;
* Move the Jira task to the **TESTING** status.