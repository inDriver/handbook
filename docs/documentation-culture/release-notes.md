# Release Notes

Write release notes to share recent updates with the teammates.

Release notes should **focus on the impact for users and teammates and make that understandable**.

Once the release is deployed, let the teammates know in the Slack channel or any other appropriate communication apps by posting the release notes.


## When to Write Release Notes?

If the update **meets any of the criteria below**, release notes are recommended.

**Upgrades**

* a deployer needs to take an action when upgrading;
* a new configuration option is added that the deployer should consider changing from the default;
* a configuration option is deprecated;
* a configuration option is removed.

**Features**

* a new feature is implemented;
* a feature is marked for deprecation;
* a feature is removed;
* default behavior has been changed.

**Bugs**


* a security bug is fixed;
* a long-standing or important bug is fixed.

<br>



**API**


* driver interface or other plugin API is changed;
* REST API is changed.

<br>



#### Release Notes Structure


| Structure | Release notes example |
| --- | --- |
| What's new on < the app name > < dd month yyyy >.  The date optionally includes the link to the last completed sprint in Jira DOC project (for internal company release notes only).  Release version of the app (if available). Here can be a short description of what have been done by a team during the sprint | What's new on Documentary Platform, 9-23 January 2023. In this release note you will learn about  the new features of the tech blog and other updates |
| Subject headers (for example, “New Features”, “Fixes”, "Key updates" etc.) | Key updates |
| Long catchy title that shortly describes the last updates in bold type | Data Rendering from a Default Branch |
| Improvement details: short description of what has been developed / fixed with a link to Jira task in brackets (the link is optional). Visual materials if they better describe the improvements. The purpose of the improvement. How to use the improvement. The known limitations of the improvement. References to the docs and / or other applicable resources | For the time being it was possible to render data only after the merge to the "main" branch . The data from a repo are rendered from any default branch of a repo ("main", "dev", etc.) |
| Credit your contributors. If teammates from other teams helped to develop a feature, the release notes are a great place to give them a shout out |   |
