# Techniques for defense API against competitive analysis


## Overview

We operate at a highly competitive market. We are well aware that competing companies investigate our API to obtain additional information about the company, orders, and users. This information could provide our competitors with a strategic advantage. The purpose of this document is to outline adequate defense mechanisms and the rationale for their implementation.

Description of the working techniques used for competitive analysis


### Step 1. Examining our API

Qualified engineers audit mobile applications, APIs, and network traffic to find any incremental IDs. For example, new users have ID 5000, and the next created user will have ID 5001.


#### Step 2. Measuring Changes in IDs

At this step, a competitor's engineer builds bots, scripts, or other automation to periodically get new values for our ID. For example, getting a new user ID every 24 hours or 30 days allows competitors to measure how fast our user base is growing over time. After obtaining the ID, a competitor's engineer writes the obtained data into the database.


#### Step 3. Reports and Conclusion

After this, the competitor's engineer will build a report to evaluate our metrics.


#### Additional Information

Some indicators may be closely related in information systems, so disclosing them permits drawing conclusions concerning other indicators.


##### 1-to-1 Linkage

An incremental ID can be linked to a UUID ID. This is the most dangerous relationship because it destroys all the advantages of UUID ID. For example, an order has a UUID ID, but each order corresponds to a record in service X, where incremental IDs are used. By measuring the ID in service X, competitor's engineers can measure changes in orders quite accurately.


##### Indirect Correlation

The number of IDs can correlate with UUIDs in a certain way. For example, the number of requests to the helpdesk may be related to the number of users or orders by a fixed multiplier.


##### Enumeration Attack

In addition to directly determining the values of IDs by disclosing their values, a competitor can exploit the behavioral features of a restricted access control system (RBAC).

For example, we have the following API: `/chat/&lt;chat_id>/`, where chat_id is incremental and API has the following behavior:

* if `chat_id` exists but the user doesn't have the right to view it, the system returns the HTTP status code 403 and a relevant response body;
* if `chat_id` does not exist, the system returns a slightly different response: a different HTTP status code or body.

This inconsistency led to the ability to determine the last actual ID in the system. The result is the same as the direct ability to read the ID value, but the attacker does not need to perform any active actions to create such IDs.


### Decision


#### Prohibition of Incremental ID

The system must not use an incremental ID for any data created by the user. Instead, you need to use [UUID v7 or v8](primary-key-selection.md). Including, but not restricting the following IDs: order id, document id, user id, support request id


#### Acceptable case of using Incremental ID

The system may use an incremental ID for the elements of the predefined system list, for example:

* City ID;
* Country ID;
* any geo-related ID;
* ID of transport type;
* backend server ID;
* payment instrument ID (such as a bank card, cash, etc).

There are usually not many of these IDs, and they are publicly available for all clients, so an attacker can easily discover them. Consequently, efforts to hide or use a UUID instead of an incremental ID do not make sense.


#### Working with Legacy Incremental ID

All legacy incremental IDs must be:

1. Submitted to the product security and architect's team via chat `#architectural-committee`.
2. Hardened against attack on the RBAC system (see below).

Information about legacy incremental ID enables the security and anti-fraud team to build a monitoring system to detect these attacks. Hardening against the RBAC attack makes it more complicated for an attacker and more visible to us. Also, an incremental ID is an architectural technical debt that can be fixed.


#### Prohibition of Virtual ID

You should not use virtual short-time living IDs because this will complicate your system. Virtual short-time living IDs may be possible only in the most extraordinary cases. The decision must be approved after a detailed consultation with the architecture and security teams.


#### Prohibition of Disclosing Internal Information

The Backend API should not disclose details about any internal errors. This prevents the disclosure of sensitive information, including business metrics.


#### Permission to use Limited and Necessary Information in Error Message

Backend API may disclose the error ID, short name, and description for the user without details, for example: Error 5009. Internal Error.

We have some internal problems, and we are already working on them. Please try to make your request again.


>**WARNING**
>
> Be careful when using a unique primary personal ID. Although not typically incremental, competitors can draw certain conclusions about the market conditions if they observe orders with a unique PI. 