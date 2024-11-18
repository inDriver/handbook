# Documentation Hubs

There are two platforms used for documentation: **Documentary Platform** and **Confluence**.


## Documentary Platform

The Documentary Platform (SaaS) stores engineering documentation. Under the engineering documentation the following types of internal tech documentation are considered:

* Documentation (`.md` files):
    * `README` files (component / service launch);
    * tech documentation for components / services (including docs for tools, libraries);
    * how-to guides;
    * `CONTRIBUTING` file on how to contribute to a repo;
    * team tech processes having impact on the general development procedure;
    * run books;
    * ADR;
    * `.documentary.toml` - a file used for connection to the platform and which includes additional data on component / service to be rendered on the platform.
* Diagrams:
    * PlantUML: sequence diagrams, ER-diagrams, system and container diagrams;
    * Mermaid diagrams.
* GraphQL;
* Swagger;
* documentation built from code comments, usually added to `.md` files by snippets;
* dependencies (dependency graph).


## Confluence

* drafts developed before the development cycle starts;
* team's internal processes;
* team's calendars;
* team's internal plans;
* MoM;
* discussions and hypothesis;
* engineering notes.