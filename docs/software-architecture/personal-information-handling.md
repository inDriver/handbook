# Personal Information Handling


## Overview

This document is intended to clarify what is Personal Information (PI) and how it must be handled within inDrive company.


### Decision


#### Personal Information Definition

* **Personal Information (PI)** — is a kind of information that relates to an individual (customer or contractor) even if it does not allow to identify them;
* **Primary Personal Information (PPI)** — any kind of PI provided by a individual to a company that relates to an individual. This includes data that can be used to distinguish or trace an individual's identity, such as:
  * full name;
  * date of birth;
  * social security number or national identification number;
  * contact information (e.g., phone number, email address, physical address);
  * financial information (e.g., bank account details, credit card numbers);
  * biometric data (e.g., fingerprints, facial recognition data);
  * government-issued identification numbers (e.g., passport number, driver's license number).
* **Secondary Personal Information (SPI)** - any kind of PI created as a result of a person's interaction with inDrive. This data is PI only in combination with Primary PI. For example:
  * user ID;
  * order history;
  * IP addresses;
  * geolocation data.

As a result, SPI must be treated as PI for as long as PPI is stored in our systems. There are no special requirements to store and process Secondary PI after Primary PI is deleted.


#### Primary PI Protection

A developer must avoid storing PI permanently in services. A limited number of services / solutions are supposed to do this:

* **user service** — the service is designed as a main storage of users PI;
* **watchdocs** — stores parsed data from documents uploaded by a user;
* **file storage** — stores users documents scans, photos and other binary data. Implements the required data encryption and landing functions
* **Payment & Payment Provider** — stores users bank account details in accordance with [PCI DSS](https://www.pcisecuritystandards.org/standards/) standard
* **DWH** — a special segment with a limited number of users must be used to store PI to fulfill legal claims. General store must avoid exposing personal data.

Personal information may be cached in other services for a limited period of time.

If you have identified a new type of personal information that needs to be processed or stored, first of all try to understand this information domain. If a new field or entity may be considered as a property of inDrive user, then user service should be considered to store such information. If it is a document, then you should consider storing it in watchdocs service. Watchdocs implements measures to protect PI in database and uses a file storage as storage backend, so you should not worry about raw document data protection.

If it is a binary data, that can't be classified as a document, you should store it in file storage directly. If you cannot classify this new type of
personal information, please contact the Architect Team for help.

If Architects Team made a decision to store PI a solution must be reviewed by Product Security team and the following measures must be implemented depending on how this data will be processed.


#### Protection of PI in Databases

* database credentials must be personalized and used by a single user only. Data owner MUST approve new credentials added;
* database user permissions must follow the Least Privilege principle;
* a long-term storage must be created to store audit logs for all PI access operations from Admin panel.

For additional hardening:

* PI data may be encrypted with a user's private crypto key;
* user's crypto key may be stored in an encrypted container created with Vault Transit Engine;
* PI data may be stored in separated by regions / countries databases.


#### Protection of Binary Data Containing PI

* if a binary data is small enough to fit into a database BLOB field, then a standard set of measures to protect PI in a database must be applied;
* otherwise, a file encryption must be implemented with the owner's private crypto key;
* user's crypto key must be stored in an encrypted container created with Vault Transit Engine;
* file names must not be a linear sequence to exclude the possibility of scanning all files one by one. The best solution here is to use UUIDv4 as file names;
* access to file data containing PI must follow the Least Privilege principle;
* if access to file data is granted via URL, it's lifetime must be limited. The shorter URL lifetime is, the better
* files containing PI must never be stored in public CDN or caches.


#### Protection of PI in Kafka

* streaming PI generally should be avoided;
* the best solution to notify subscribers about PI changes is to send change notification events;
* on such event a subscriber must invalidate existing entry in cache and lazily get the updated data with synchronous request to PI source;
* if streaming PI cannot be avoided, separate topic must be created for this purpose;
* permissions to read data in this topic must be strictly limited with RBAC;
* each service, consuming personal data from Kafka, must use a dedicated account.


#### Protection of PI in Application Logs

* personal information in logs must be wiped out totally for all new services;
* if there is a need to identify PI data in logs, each field must be masked in accordance with the guide developed by the Application Security team.

> **NOTE**  
> 
> A special case of PI is a **user avatar**. By definition, this must be treated as Primary PI, but we understand the complexity of technical measures for data protection and agree, that avatars must be stored with simplified protection:
> 
>  * only authenticated users may access other users avatars;
>  * there is no need to encrypt avatar with user's personal encryption key.


#### PI Erasure

App marketplaces and data protection laws require that we delete user data upon expiry of legal retention period.

A special solution to implement and control automatic PI erasure - the User Termination service is implemented.

A users`s account will be deleted in two stages:

* user deactivation - the account gets suspended by user request or due to inactivity, but all personal information is kept for the terms defined by collected data type. Account restoration SHOULD be impossible at this point. PI Retention periods:
  * six months if the user did not make rides or use any services;
  * three years if the user made rides or used services (excluding financial);
  * ten years if the user made payments or used inDrive financial services.
* personal data deletion - after the retention terms are finished, user`s Primary PI data MUST be completely wiped out of all inDrive systems by a delete operation, one-way hashing or by setting a tombstone.


#### PI Tagging in Databases of Production Services

For every production database there is a requirement for it's `INFORMATION_SCHEMA` to contain special tags indicating the type of PI it stores. This requirement is only for production databases.

Developer of a production service which stores PI in SQL Database is responsible for tagging columns and tables.

Tagging is the process of placing PI tags which contain personal information in the comments of database columns and tables.

Two types of tags should be placed on these columns and tables:

* `#PPI` — indicates that database or column contains Primary PI;
* `#GEO` — indicates that database or column contains geolocation data.

If a comment field of a column or a table already contains some other text, tags should be added after that text.

A developer MUST add `#PPI` and `#GEO` tags to comments of initial table / column CREATE statement and maintain persistence of those tags during the process of migrations.
