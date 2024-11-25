# Parsing Protection Requirements.


## Overview

There are several major verticals within inDrive, each with its own order feed, its own workflow, and a set of unique methods. The presence of excessive personal information (PI) in the API, as well as the lack of configured rate-limits, can lead to mass parsing and therefore possible leakage of user data.

So each company is at risk of a data breach, no matter how small or large a company is, there are several examples:

* Facebook (leakage 530M users data included phone, account names, and Facebook IDs) - it cost $277M;
* Yahoo (breach of 500M accounts including names, email, phone, and dates of birth) - $85M payout;
* Uber (PI breach of 57M users and drivers which contains names, email, and phone) - paid the hackers $100,000 and then was fined $148M.

Consequences of Mass Data Leakage for Businesses:

* financial loss;
* reputational damage;
* increased regulation.

This ADR is designed to minimize the risk of leaks in sensitive API methods that are susceptible to parsing.


## Options

Personal Information (PI) — is a kind of information that relates to an individual (customer or contractor) and allows to identify them (see [personal information handling](personal-information-handling.md)).

Each of these order feeds uses personal information that can be automatically collected by attackers in large volumes. To protect users' PI, it is necessary to exclude all optional data from the API response from the point of view of business metrics, as well as to apply measures to reduce the motivation for parsing (data filtering, rate-limits, shield_id verification, etc.). Also using statistical data (number of orders, geo-positions of contractors, etc.), the attacker can estimate the approximate volume of the vertical in the region.

Let's look at the 2 main causes of leaks:


### Excess PI

The problem occurs when the back-end returns more data to the front-end than necessary, for example in the flow of placing an order in a feed when the order contains users PI that are not needed for the display and functioning of the order feed:

* phone number of customers / contractors, before accepting the order;
* plate number of drivers;
* full address of customer.


### No Limits

Insufficient attention is paid to the rate-limits when developing APIs, which leads to the possibility of parsing. The most obvious example is the lack of limits on the number of orders created and canceled by 1 client.

In addition to the two main causes of leaks described above, which are obvious business risks, there are other problems that cause financial and reputational losses:

- activity of bots and modified unofficial versions of the inDrive app;
- fraud attacks on billing functionality, such an example is an attack on receiving and sending OTP code via SMS;
- unlimited collection of information on the actual geo-position of contractors and customers.


### Decision 

Security requirements were compiled based on the security audits and the identified vulnerabilities which allowed attackers to parse sensitive methods in APIs. So there are some examples such sensitive functionality:

* creating and modifying orders in all feeds;
* workflow of accepting orders;
* methods that use dynamic geo-positioning to track contractors
* order history.

Where requirements are unclear or an exception is required to implement business requirements, you must consult with the product security team to discuss your case.

General security requirements for APIs that work with PI:

| Requirement                                                                         | Description |
|---------------------------------------------------------------------------------------| --- |
| Must no excess fields containing customer / contractor PI                             | Plan in advance which fields and PI are necessary to avoid excess; the back-end should return only the minimum data required by the front-end |
| Should availability of full L7 logging for anomaly detection and incident investigation | If possible, log all events at the HTTP level with the following fields: timestamp, `user_id`, path, IP address, user-agent, request URI with query parameters |
| Should make critical data retrieval, such as phone numbers, a separate method         | If possible, place the retrieval of critical PI in separate methods to enable more precise rate limiting |
| May use Captcha                                                                       | Captcha is effective for protecting against scraping, but should be aligned with business requirements. It may not be suitable everywhere, as it can impact conversion rates and incur high subscription costs for high-load systems |
| May implement integrity API (consult with AntiFraud Team)                             | Integrity API provides basic protection against scraping and fraud. Consult the AntiFraud Team if you have questions about implementation |
| May add `shield_id` check (consult with AntiFraud Team)                               | Bots or parsers may lack a `shield_id` or have a low trust level, which can be used to block illegitimate traffic. Consult the AntiFraud Team for details |
| May use request signature                                                             | Pre-sign sensitive requests on the front end using a key stored on the front end to make scraping more difficult. Self-signing is a temporary solution and is recommended for requests involving company billing, such as OTP SMS requests |

Implementing the following measures and constraints is necessary to make parsing in order feeds more difficult:

| Requirement                                                                   | Description                                                                                                               |
|-------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------|
| Must order PI is disclosed only to the customer and contractor, after biding flow | Order PI are only available for customer/contractor, who make and accept this order, NOT for the other customer / contractors |
| Must accepting orders only for confirmed contractors                          | Only those contractors who have passed an identity check and provided the required valid documents can get access to orders |
| Must filtering orders by geo-distribution                                     | Having a geo-distributed restriction, i.e. orders in the feed are controlled by the coordinates set in the controctor profile |
| Should filtering by order status                                              | Only orders in active status are shown in the order feed                                                                  |
| May access to view orders in the feed for confirmed contractors               | Access to view orders in the feed can only be granted to confirmed contractors                                            |
| May cell number spoofing and chat instead of giving out a real phone number   | If possible, refuse to give out a real phone number and switch to chat                                                    |

Implementing rate-limits aimed at making parsing more difficult:

| Rate-limit                                                              | Description |
|-------------------------------------------------------------------------| --- |
| Must rate-limit on the number of valid requests by `user_id`            | For example, on the number of active or canceled orders for a certain period of time like 3-6-12-24 hours etc. Such limits are set only after agreement with the business |
| May implement limits per IP / user-agent or shield_id (any combination) | Low cost limits which can help to mitigate mass parsing |
