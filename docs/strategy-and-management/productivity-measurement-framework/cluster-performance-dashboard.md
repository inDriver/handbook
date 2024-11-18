# Cluster Performance Dashboard


## Overview

The Cluster Performance Dashboard for product teams provides a comprehensive view of key performance metrics of all product clusters, organized into several categories to enable efficient monitoring and analysis. This dashboard is designed for cluster heads (HoE, HoP, Agile Coach) and other stakeholders who need to track the progress and performance of clusters in alignment with organizational goals.


## Technical Details


### Access to Dashboard

The current dashboards are available on the tableau system. To access Tableau, follow these steps:

* to use dashboards on Tableau Server, it is necessary to configure VPN beforehand; 
* request access to Tableau by submitting a request through the service portal on the Teammate.


### Data Sources

The dashboard pulls data from other dashboards and various internal systems and tools used within the organization. If a dashboard pulls metric data from another dashboard, the widget description will include a link to the master dashboard.


### Troubleshooting

If any data appears to be incorrect or missing, users are advised to contact the support team by writing to Slack channel `#team-performance-dashboard`.

Any changes to the list of dashboard metrics, their calculation methods, or visualization can only be made through a request to the BI team and after approval from the responsible PMO team.


## Key Features

Top widget provides a summary of key metrics, offering an overview of the cluster's performance across several critical areas:

* **Predictability:** displays metrics like Current Quarter OKR Progress and Scope Drop, helping to understand how well the cluster is meeting its plans and commitments;
* **Speed:** reflects the efficiency of task completion, measured through metrics like Lead Time and Velocity, allowing for an evaluation of the team's productivity;
* **Quality:** includes indicators such as Tech Debt Index and Security Error Budget, which help assess the quality of code and the level of technical debt;
* **Maturity:** shows the maturity according to the Team Maturity Model and divided by Tech TMM, Product TMM, and Process TMM. This widget is a central component of the dashboard, enabling users to quickly gauge the cluster's key performance indicators and identify areas that may require attention;
* **Extended widgets** - below the main widget, there is an extended metrics section with other widgets by metric, where the same key metrics are available for more detailed analysis. In this section, users can explore these metrics over time, with a breakdown by teams, enabling a deeper understanding of trends and team-specific performance.


## User Guide

* **Filters and Settings:** users can select the cluster, teams that belong to the cluster, period for which they want to view the data. By default, widgets display consolidated metrics for all teams within the cluster for the recently completed sprint;
* **Interpretation of Metrics:** under each widget, there is a brief description of the metric along with actionable insights. These descriptions provide guidance on what decisions or actions should be considered based on the data presented. This helps users not only understand the metric but also know how to respond when performance is suboptimal.
  
  Additionally, when hovering over the icon next to a metrics name, a tooltip appears with a detailed explanation of how that metric is calculated. This feature ensures that users have a clear understanding of the underlying methodology, enhancing transparency and accuracy in data interpretation.<br>On this dashboard color coding is used to visually represent how each metric compares to its target value. This visual aid helps users quickly identify whether the cluster's performance is on track or if there are areas that require immediate attention.
* **Green:** indicates that the metric is within or exceeds the target value. This color signifies that the cluster or team is performing as expected, and no immediate action is required;
* **Yellow:** shows that the metric is below the target value but not in a critical range. This serves as a warning that performance may be slipping and could require monitoring or corrective action to bring it back to the target;
* **Red:** denotes that the metric is significantly below the target value and has entered a critical zone. This indicates a serious issue that needs urgent attention and corrective measures to address the underlying problems.

This color-coded system allows for a quick understanding of the cluster's performance across various metrics, making it easier for managers to prioritize actions and make informed decisions.

If there is no target value for the metric, color interpretation is not used.


## Widgets Description & Target Value


### Predictability


* **Current Quarter OKR progress** shows the current quarter OKR progress as an average, using data from all teams within the cluster. This helps track progress toward strategic goals, identify areas needing attention.
* **Target value:**  >70;
* **Color coding:** green >70,  yellow 40-69, red  <40.
* **Scope Drop** shows what proportion of the initially planned scope was excluded from the sprint. This helps to understand how accurately the team plans its tasks and manages the scope of work during execution.
* **Target value:** <20%;
* **Color coding:** green <20%, yellow 20-40%, red - >40%;
* **Formula:** (Plan SP - (completed from plan SP) / (Plan SP).


### Speed

* **Lead time**. This metric provides transparency and understanding of the time required to deliver tickets from start to finish, including wasted time. This helps identify bottlenecks and improve efficiency within the cluster, while allowing performance comparisons to identify best practices and areas for improvement.
* **Target value:**  <7 days;
* **Color coding:** green <7 days, yellow 7-10 days, red - >10 days;
  
    **Formula:** Delivery time + Wasted time, issuetype = story, enabler, BUG;
* **TTM**. The purpose of the Time to Market (TTM) widget is to measure and track the total duration from the initial planning stage to the launch of each UPD idea. This helps identify bottlenecks, optimize workflows, and accelerate value delivery, ensuring products meet market demands promptly and maintain competitive advantage. 
  **Formula:** TTM = sum of time in statuses in UPD: planned, problem research, solution research, PBR, ready for development, delivery, experiment scheduled, experiment started, impact verification, launch scheduled, and launch started;
* **Velocity**. This widget provides a visualization of performance by measuring teams work speed in the sprint. You can use this dashboard as a tool for tracking progress, identifying deviations, comparing different teams that can help to make improvements in capacity planning, identifying trends, identifying bottlenecks, improving collaboration, risk management etc. 
  
    **Formula:** Velocity = Number of completed story points/issues per sprint


### Quality

* **Quality Rate**. Widget assesses the quality of work delivered during a selected time period and provides insights into the effectiveness of development and testing processes. It helps identify areas of concern, such as frequent high-severity bugs, and supports corrective actions to improve product quality. Additionally, it tracks quality improvements over time, enabling data-driven decisions to enhance cluster performance and ensure high-quality software delivery.
* **Target value:** 100%;
* **Color coding:** green >98,5, yellow 95 - 98,4% ,red - <95%;
* **Formula:** the metric is calculated using the following formula over a rolling period of 90 calendar days: "100% - 100 * (Bug Points / Tested Tasks)%". Bug Points = 5 * Critical Bugs + 0.5 * High Bugs + 0.1 * Normal Bugs + 0.05 * Low Bugs.
* **Tech Debt Index**. The widget visualizes the Tech Debt Index over a selected period, showing overall and documentation-related technical debt for each team in the cluster. The diagram shows the total amount of technical debt in the backlog, the number of techdebt issues completed overdue and the number of techdebt issues closed within the sprint.

Its goal is to clarify technical debt distribution, helping identify teams that need to focus on debt resolution to maintain a sustainable codebase.

**Formula:**

* **% of technical debt related to documentation** = the number of enabler issues with Backlog Type[Dropdown] set to TechDebt, labeled as documentation, and resolved as Done in the sprint, divided by the total number of enabler issues in the team labeled as documentation.
* **% of technical debt not related to documentation** = similarly but excluding issues labeled as documentation.

Overall Tech Debt Index = average between the above values.


### Maturity

The "Team Maturity Model Index" widget displays the maturity level of teams in terms of processes, products, and technologies according to the Team Maturity Model. It allows tracking the progress of teams and clusters across key aspects of their development, helping leaders make strategic decisions to improve efficiency.

* TMM Index - overall index;
* Tech TMM Index - technical cards index;
* Prod TMM Index - product cards index;
* Proc TMM index - processes cards index.

The data in the dashboard is updated quarterly after the TMM review and collected from Master TMM dashboard.