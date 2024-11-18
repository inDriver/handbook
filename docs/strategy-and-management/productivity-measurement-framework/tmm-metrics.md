## Team Maturity Big Picture

Team Maturity Big Picture is a tool that represents the consolidated picture of the team maturity: the combination of self-assessment by Team Maturity Model and metrics for the quarter. 

Now we use only TMM Data and plan to build a up-to-date informational center integrated with all metrics that show team maturity level. 


### Access to the Dashboard

The current dashboard is in [Tableau](https://tableau.console3.com/#/views/MaturityBigPicture/byTeams). 

See the [detailed instruction](https://drive.google.com/file/d/1m10vLt1z75zgyirz4q52SHGTmvREurL2/view) on how to get access to Tableau.


### How the Dashboard Works


#### First Page

The data is pulled from the [spread-sheet on the results of filling out the Team Maturity Model](https://docs.google.com/spreadsheets/d/1TH9iJ7DNGAB-_tpu0adtRYSXCimiUlOQiiud6vSulK4/edit#gid=2129602632).

The data is distributed by:

- teams,
- clusters,
- division.

It is possible to filter data by clusters and individual teams.

The data are stored in the form of a pop-up window when hovering over the level. The level is counting as average of all TMM question cards that connected to Tech, Process or Product area. 


#### Second Page

Under development. 

The idea of the second page is to show the development of one choosen team. We show:

- maturity dynamics by card or how every card was changed,
- overall maturity and how it is according to target value,
- target actions or cards that a team has in its backlog.


### Dashboard Update Schedule

The data is updated once a quarter, at the same time you can add additional teams, update the mechanics of calculating metrics, etc.


### When Use the Dashboard

Once every four months after cutting by Team Maturity Model. It is important to track the trends of team development and the overall level of maturity by clusters/division.


### Metrics Calculation


#### First Page

The dashboard calculates the average for the group of cards (quality assessment and level to which team metrics fall):

- tech,
- process,
- product.

Also in the dashboard are displayed Tags, which are the team's focuses for the current period. It takes from the separate page in spread sheet. 


#### Second Page

Under development.

We use [spread-sheet data](https://docs.google.com/spreadsheets/d/1TH9iJ7DNGAB-_tpu0adtRYSXCimiUlOQiiud6vSulK4/edit#gid=2129602632) to show the maturity dynamics. Also we use Jira data as team name, borad key to map tickets and teams in dashboard; label 'TMM' to find TMM-related tasks in the backlog in Jira; target actions choosen by teams from spread sheet. 
