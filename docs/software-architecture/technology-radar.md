# Technology Radar


## Overview

In our dynamic technology landscape, maintaining a curated and standardized set of languages, technologies, and tools across all of inDrive is crucial. To support this effort, the Technology Radar should serve as the single source of truth for guiding technology decisions. This centralized reference will ensure alignment, consistency, and informed choices in adopting and evolving our technology stack.

inDrive tech radar is available via the [link](https://indriver.github.io/architecture-docs/). 

> **NOTE**
>
> The Technology Radar is a well-regarded resource that categorizes technologies into `Adopt`, `Trial`, `Assess` and `Hold` categories providing clear guidance on their suitability for organizational use. This radar not only reflects industry best practices but also captures our collective knowledge and experience in the ever-evolving technological ecosystem.


## Decision


### Adoption of the Technology Radar

The Architectural Vision Working Group unanimously decided to adopt the Technology Radar as the definitive reference for technology choices within inDrive.

This decision underscores the importance of maintaining a standardized set of languages, technologies, and tools across the enterprise. The Technology Radar must serve as the authoritative guide. It should reflect our collective knowledge and experience in the ever-evolving technological landscape.

Emphasizing stability and reliability, critical production services MUST exclusively rely on tools and technologies classified as `Adopt` to provide stable SLA/SLOs.

For the majority of remaining services only tools and technologies categorized as `Adopt` and `Trial` should be used. None of new services should use technologies or tools categorized as `Hold` or `Decommission`.

Teams may propose new solutions during usual Architecture Committee events or during special dedicated Architecture Committee just for this solution or technology, initiating an incremental adoption process. Such new technologies if they have good potential and approved by Architecture Committee Board Members may then enter an assessment phase and added into the `Assess` category, which initiate the process of adoption of new tool or technology.

> **CAUTION**
>
> Strategic overrides may be considered only in exceptional cases, subject to thorough justification and approval.


### Technology Radar Processes

The evolution of the Technology Radar is governed by a structured process, taking into account the dynamic nature of technology and the need for careful consideration in adopting new elements.

The Technology Radar, a YAML/JSON file containing a list of technologies, utilities, and programming languages along with their usage levels and brief descriptions, resides in the architectural repository `architecture-docs`. It is rendered into an interactive widget using the Documentary system, mirroring the format
of https://radar.thoughtworks.com/.

The responsibility for making changes and maintaining the currency of the Technology Radar lies with the architecture team. All changes in the Technology Radar must be done via Pull Requests to the repository.

During Architecture Committee sessions, each team has the opportunity to nominate new technologies, services, or solutions not yet included in the Technology Radar. If the merits of a proposed solution are evident and appear promising, the Architecture Committee may categorize it as "Assess" and add to the Technology Radar.

When adding a technology to the "Assess" category, careful consideration is given to the availability of resources for its deployment, both from the development team and the platform / support / monitoring perspectives. If there are not enough resources, the solution is not added to the "Assess" category and should be replaced with comparable solutions already present on the Technology Radar.

Elements in the "Assess" category are only allowed on production servers in a limited capacity, utilizing Feature Toggles or MVPs, to validate claimed capabilities and confirm positive attributes on a small and restricted user segment.

Usage of any element from the "Assess" category within critical production services is strictly prohibited, as it may lead to failures and violation of SLA/SLO metrics established for such services.

After a minimum of one month, a solution can transition from the "Assess" category to the "Trial" category upon receiving positive feedback from the implementation / support and monitoring teams regarding its stable performance.

However, solution / technology has high limit of maximum six months to complete transition from the "Assess" category to the "Trial", otherwise it will be forced to move into the "Hold" category as we haven't received positive effect from it
and should stop allocating our resources on it (for example, cases with Vitess or YDB).

Quarterly (or upon request), a Technology Radar audit is conducted to ensure the relevance of the list of services / technologies. Services in the "Trial" group that have been in this status for more than two months are recommended for promotion to "Adopt". Services not actively used in the last two months are moved to
"Hold" and remain there for a minimum of two years.

"Decommission" state is used to mark any existing technology or service unwanted on production due to different circumstances (serious vulnerability issues, lack of support, quick change of licensing like Java or Docker). Service may continue to operate for a short period of time and there should be active ticket to resolve this Tech Debt. Transition to this state is possible from any other state, including "Hold".


### Technology Radar Committee Board

Technology Radar Committee Board is not fixed, but must always include representative members from following teams:

Architecture Team, Infrastructure Team (including Cloud and Database teams), Security Team, Observability Team, Data Department (DWH) and Mobile Platform teams.
Presence of (HoE) Head of Engineering from all clusters is recommended as well to be aware about new technology. Each HoE has the same voting right as all major members of the Technology Radar Committee Board.

Technology Radar Committee Board must use consensus mode of Voting (50% + 1 vote) for making a decision.

There are pre-defined time-slots for the Technology Radar Architecture Committee, thus existing weekly Architecture Committee slot should be used for this.


### Technology Overview

In order to reduce the overall time, which might be needed for technology evaluation during the Architecture Committee event, following important questions may be covered:

* is this new technology, tool or solution Cloud-Native? Check https://www.cncf.io/projects/;
* what is our expected positive impact from this new technology / solution? On TTM? On GMV? On SLA?;
* how much effort will be required to start using it? Check if codebase is well-maintained, well-documented, uses open standards or our existing technologies (eg. Helm, K8s, etc);
* do we have enough resources (including human, financial and technical) to run effectively this new technology or solution?;
* what are the Strong and Weak points of this new technology or solution? Compare with our existing technologies;
* can it be reused in many services, or it suits only for limited set of services?


### Technology / Service in Use Meaning

Main indicator of any technology / service being used — its presence and active usage on production. So, any technology or service is in use, if it presents and used on production.

Technology / Service marked as a "Hold" if it was previously used on production (eg "Adopted" or "Trial"), but now we do not want any new services with such technology, however, we also don't need existing production services migrate asap
to new solutions. For example, usage of NATS in our new services - we would like to have Kafka, but it is ok to use it on production for existing services.

Technology / Service can be moved from "Trial" to "Adopt" only after 2 months of stable running on production environment. This time is used to collect general information about usage of such service, to improve operational excellence.

Any state can transition to the "Decommission" state, which means that we have urgent external force to remove existing service from production (eg, security issues, licences, financial requirement, etc.). Service should stop using
this technology as soon as possible.


### Rationale

The introduction of the Technology Radar aims to achieve the following major goals:

* standardization and curated technology stack: ensure a curated and standardized set of languages, technologies, and tools across the organization by leveraging the Technology Radar as a central reference;
* reliable decision-making: provide a reliable source of truth for technology choices, guiding teams towards the adoption of proven and battle-tested solutions, particularly for critical production services;
* incremental adoption process: facilitate an incremental adoption process, allowing teams to propose and evaluate new technologies through Architecture Committee events, ensuring systematic and well-evaluated introductions;
* resource optimization and governance: optimize resource allocation by considering the feasibility and availability of resources for the deployment of new technologies, and enforce governance to prevent the use of untested solutions
  in critical production services.