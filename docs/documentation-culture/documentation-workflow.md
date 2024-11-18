# Documentation Workflow


## Docs-as-Code Approach

The reasons we use 'docs-as-code' approach:

* the same version control system for devs code and docs code — no switching between tools when documenting;
* the same release workflow for docs and devs code;
* inclusion of docs in code review;
* tracking of doc mistakes as bugs;
* accurate tech docs content due to close collaboration of devs and tech writers;
* automated testing (markup language and spelling testing);
* automated building of beautiful artifacts in any required forms without formatting;
* continuous delivery of all tech docs code output to any platform and any converted format without much human intervention;
* use of docs metrics.


## Documentation Positive Impact on Engineering

Effective documentation plays a crucial role in engineering reducing time to market, and this relationship can be measured through various metrics. Here are some key points on positive documentation impact:

1. **Speed of onboarding (metric: time to productivity)**: high-quality documentation helps new hires quickly understand the project’s framework, tools, and best practices. This reduces the time it takes for them to start contributing effectively.
Time to productivity measures the time from when a new team member joins to when they become fully productive.
2. **Efficiency in development (metric: development cycle time)**: clear and comprehensive documentation of requirements, architecture, and code ensures that engineers can work more efficiently, reducing misunderstandings and rework. 
Development cycle time is the time taken from the start of development to the final release. Well-documented projects typically see shorter cycle times.
3. **Error reduction (metrics: defect density, rework time)**: documentation reduces the likelihood of errors in code and misunderstandings of requirements, which in turn reduces the number of defects.
   * defect density: measures the number of defects per unit of code (e.g., per 1,000 lines of code);
   * rework time: the time spent fixing issues that arise due to misunderstandings or poor initial implementation.
4. **Higher decision-making (metric: decision time)**: well-documented decision processes, historical decisions, and guidelines allow teams to make faster and more informed decisions.
Decision time measures how long it takes to make key decisions in the project, which can directly impact the speed of development.
5. **Collaboration and communication (metrics: communication amd collaboration index)**: documentation provides a shared reference point, reducing the need for repetitive communication and improving collaboration across teams.
   * communication index: the delay between queries and responses within teams;
   * collaboration index: a composite metric that could include factors like the number of inter-team meetings, email chains, or Slack messages needed to resolve issues. Lower values indicate more efficient collaboration.
6. **Consistency and standardization (metrics: compliance rate and standardization index)** documentation helps ensure that all team members follow consistent coding standards, design patterns, and operational procedures, leading to fewer errors and smoother integration.
   * compliance rate: the percentage of code or processes that adhere to documented standards;
   * standardization index: a measure of how uniformly practices are applied across the project or organization.
7. **Knowledge transfer (metric: knowledge retention rate)**: proper documentation facilitates the transfer of knowledge within the team, ensuring that critical information is retained even when team members leave or transition roles.
Knowledge retention rate measures how well critical knowledge is preserved over time, often assessed through surveys or the ability to continue work without specific individuals.
8. **Risk mitigation (metric: risk exposure)**: documentation helps identify and mitigate risks by providing clear guidelines on handling potential issues, thereby reducing delays caused by unforeseen problems.
Risk exposure measures the potential impact of risks on the project timeline. Better documentation usually correlates with lower risk exposure.
9. **Customer support and satisfaction (metrics: customer support response time and customer satisfaction score)**: documentation helps both internal teams and end users by providing clear instructions and troubleshooting guides, reducing the time needed to resolve issues and increasing customer satisfaction.
   * customer support response time: the average time it takes for support teams to respond to customer inquiries;
   * customer satisfaction score (CSAT): a measure of how satisfied customers are with the product, often linked to the clarity and availability of documentation.
10. **Scalability and maintenance (metrics: scalability index and maintenance time)**: well-documented systems are easier to scale and maintain, as new features can be added and bugs fixed more efficiently.
    * scalability index: a measure of how easily the system can be scaled based on the clarity and organization of its documentation.
    * maintenance time: the average time taken to perform routine maintenance or updates.
11. **Regulatory compliance (metric: compliance time)**: in regulated industries, documentation is essential for proving compliance with legal and regulatory requirements, avoiding delays due to non-compliance.
Compliance time measures how long it takes to prepare and submit necessary documentation to regulatory bodies.


## Tools and Language

* editor environment — any editor tool supporting Markdown and relevant plugins;
* docs format — lightweight markup language Markdown.

> **NOTE**
>
> For Markdown "flavors" it is required to analyze whether these elements can be parsed and rendered on the platform.
>
> The available Markdown elements are specified in the ["Markdown Guideline"](./markdown-guideline.md).

* docs language — **the English language**;
* publication hub — the Documentary Platform (SaaS) (for more details read [here](./documentation-sources.md)).


## Workflow

This workflow is applied for engineering documentation and requires synchronous update with the updates in codebase.

The workflow includes the following stages:

* documentation update:
    * **new features**: documenting of newly developed features and merge to the default (main) branch with code updates;
    * **general improvements**: working with documentation tech debt, docs refinement, typo corrections, error fixes and better explanations through clearer writing and more examples.
* review and addressing comments feedback (if any);
* deploy with the code update.

Contributors and reviewers are done by the owners of documentation.


### Documentation Ownership

The documentation ownership is required to address promptly to a page / documentation owner.

The owners of the documentation are specified in a `CODEOWNERS` file located in the repository root. If owners are not specifically mentioned in the `CODEOWNERS` file, the reviewers are the owners of the component / service by default.

Within one repository, ownership can be specified for separate `.md`  files as well as for entire documents. The specified owners are rendered on the Documentary Platform.


### Updating Documentation

> **NOTE**
>
> Before contributing to a repo, it's better to consult with a `CONTRIBUTING.md` file or with a repo's owner.

Generally, update of documentation in a repo is the following: 

1. Create a branch off of a default (main) branch. Specify a Jira issue tag in the branch name.
2. Update `.md `files located in a `/docs` directory or in a repo root;
3. Make a commit message with updated docs. A commit description body, explaining why updates are being made, is optional.
4. Create a PR. The PR shall be **one** for updated devs and docs code, if the updates in the code relate to the docs.
5. Assign the PR for review. When submitting the PR for review, a reviewer shall be assigned in the **Reviewer** field of the PR.


### Reviewing Updates

> **CAUTION**
>
> All PR for `.md` files must be approved by an assigned reviewer before merging.

The documentation review is the following:

* a reviewer is assigned to a PR and notified;
* the reviewer makes line-by-line comments, if any, and requests changes to be done by the author following the comments. For more GitHub PR review details see [here](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/about-pull-request-reviews). Normally, there should be only one round of review.


#### Review by an Engineer

When an Engineer is assigned for review, it's expected that he or she is able to: 

* ensure that the information in a PR is valid;
* ensure that the information in the PR covers the updates made in the codebase.


#### Review by a Tech Writer

A tech writer reviews the updates taking into account the documentation writing requirements and templates provided for the engineers. 


##### Tech Writer's Role

Within the 'docs-as-code' approach the Tech Writer's role includes the following:

* docs update planning / coordination with the teams;
* maintaining of:
  * docs templates;
  * style guides;
  * vocabulary lists;
  * information typing and topic maps;
  * data architecture (structuring) on the devs platform (the Documentary Platform);
* assistance in docs content creation;
* review PR with docs changes also providing sufficient recommendations on the docs content and structure;
* docs content refinement;
* docs quality assurance and verification;
* 'docs-as-code' approach scalability;
* training and education;
* documentation process improvement.

Despite the fact that the devs are responsible for the docs update, the Tech Writer also assists the devs teammates in content creation especially in case of emergency.


#### (WIP) Linters


### Addressing Feedback

When a PR is returned with comments, a contributor shall:

* edit the docs following the comments;
* resolve the comments.


### Deploying Updates

Once comments are fixed, documentation updates can be merged to a repo.

> **NOTE**
> 
> Documentation is a mandatory part of the definition of done (DoD) of engineering tasks which means that each new updates to a component / service should be covered with documentation. 




