# API Guideline

The inDrive company's API must conform to the REST architectural style. A set of rules, standards, and conventions applicable to the design of RESTful APIs has been developed to achieve the goals of unification and standardization.

Every vertical service interacting with client applications must comply with the rules of this document. Compliance must be ensured through the use of a linter (e.g., Spectral) in Git hooks, as well as in CI processes.


## OpenAPI Specification Requirements

Each application with REST API **must** provide OpenAPI specification. 

* path to the specification **must** be set up in `.documentary.toml`, example:
  
  ```text
  [documentary]
  apispec = ["./api/swagger.json"]
  ```
  
* specification **must** be actual and correspond to the current API version.
* specification **must** provide the following information:
    * request methods and paths;
    * authorization methods;
    * request parameters;
    * response codes and content;
    * endpoint description.
* application **must** provide instruction `make generate-open-api`, to keep code and specification in sync.
    * for "design-first" approach this command generates code from specification;
    * for "code-first" approach this command generates specification from code;
    * example how to make an alias to existing target in Makefile:

      ```text
      .PHONY: generate-open-api
      generate-open-api:
            $(MAKE) swagger
      ```
      
* recommended version is [3.x](https://spec.openapis.org/oas/latest.html), but 2.0 (swagger) is also acceptable.

There is no restriction on using design-first or code-first approach, it's on the team to choose. 
But we highly recommend to use API-first approach, especially in a new project, due to reasons, that are described in this [article](https://learn.openapis.org/best-practices.html).

For the code-first approach we recommend to use [go-swagger](https://github.com/go-swagger/go-swagger) library. It generates OpenAPI specification v2.0 (a.k.a swagger) from code annotations.
Here you can find [annotation examples](https://github.com/go-swagger/go-swagger/tree/master/fixtures/goparsing/petstore) as well as in dev-onboarding application. Also, in dev-onboarding you can find [command](https://github.com/inDriver/dev-onboarding/blob/main/Makefile) to generate specification.

### URL Structure

URLs must be readable.

Example of a readable URL - `https://indriver.com/api/new-order/v1/orders/456` where:

1. `new-order` — namespace logically separates different APIs, which may be developed by different teams and balanced across different services. The namespace can be singular or plural and defines the scope for resources.
2. `v1` — major version of the API, which separates incompatible versions.
3. `orders` — resource collection.
4. `456` — identifier of a specific resource in the collection.

Example of an unreadable URL:

```text
https://site.com/Users('jacks@gmail.com')/Folders('AAMkADdiYzI1MjUzLTk4MjQtNDQ1Yy05YjJkLWNlMzMzYmIzNTY0MwAuAAAAAACzMsPHYH6HQoSwfdpDx-2bAQCXhUk6PC1dS7AERFluCgBfAAABo58UAAA=')
```

## Resources

### Avoid Using Verbs in Paths

The fundamental building blocks are resources, around which the API is designed. Standard `HTTP` methods indicate the operation to be applied to the corresponding resource.

Bad:

```text
POST /cancelorder/{order-id}
```

Good:

```text
PATCH /orders/{order-id}
```


### Name Collections with Plural Nouns

Nouns in the plural form must be used to denote a collection of resources.

Bad:

```text
GET /order/{order-id}
```

Good:

```text
GET /orders/{order-id}
```


### IDs in Paths Should Relate to the Resource Itself

If there is an ID in the path, it should relate to the parent resource.

Bad:

```text
GET /orders/{comment-id}
```

Good:

```text
GET /orders/{order-id}
```


### Resource Nesting

An individual resource may follow a collection:

```text
/orders/{order-id}
```

A collection can follow an individual resource as the next resource (subresource):

```text
/orders/{order-id}/drivers
```

However, the use of an individual resource further down the hierarchy is prohibited. The resource definition must be moved up a level.

Bad:

```text
GET /users/{user-id}/payments/{payment-id}
```

Good:

```text
GET /user-payments/{payment-id}
```


# Response Codes

The most common response codes are presented in this section. These codes should be used developing a new API. Codes that are not listed in this guideline may be used only if the current codes are insufficient.


## Successful Request


| Code | Value                                                                                                                                                                                                                   | Methods                                         |
|------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------|
| 200  | OK — standard code for a successful operation                                                                                                                                                                           | All                                             |
| 201  | Created — returned when an entity is successfully created, along with a representation of the created entity                                                                                                            | `POST`                                          |
| 202  | Accepted — returned when the request is successfully accepted for processing, but the processing has not yet been completed. The response may contain a link to a resource that will contain the status of the request. | `POST`;<br/> `PUT`;<br/> `PATCH`;<br/> `DELETE` |
| 204  | No Content — the request was successful, and there is no response body                                                                                                                                                  | `DELETE`                                        |


## Client Request Errors

| Code | Value                                                                                                                                                                                                          | Methods |
|------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------|
| 400  | Bad Request — used when the request body does not pass server validation logic                                                                                                                                 | All     |
| 401  | Unauthorized — the request is not authenticated (e.g., JWT token is missing)                                                                                                                                   | All     |
| 403  | Forbidden — authorization issues (e.g., the user cannot view the status of an order they do not own)                                                                                                           | All     |
| 404  | Not found — the resource was not found                                                                                                                                                                         | All     |
| 409  | Conflict — conflict (e.g., the user attempts to create an order that already exists)                                                                                                                           | All     |
| 412  | Precondition failed — This happens with conditional requests on methods other than `GET` or `HEAD` when the condition defined by the `If-Unmodified-Since` or `If-Match` headers is not fulfilled.             | All     |
| 413  | Request entity too large — used when the request body is too large                                                                                                                                             | All     |
| 429  | Too many requests — used when the client sends too many requests, and the request is blocked by the rate limiter. `Retry-After` header specifies the number of seconds to wait before making the next request  | All     |


## Server Errors

| Code | Value                                                                                                                                                                                                                          | Methods |
|------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------|
| 500  | Internal Server Error — a server error that prevents the request from being fulfilled                                                                                                                                          | All     |
| 501  | Not Implemented — the server is unable to service the request at the moment, implying future availability                                                                                                                      | All     |
| 503  | Service Unavailable — the server is closed for maintenance, the HTTP header Retry-After contains the expected recovery time                                                                                                    | All     |
| 523  | Origin Is Unreachable — An error on an external resource relative to the server prevents the request from being processed. For example, a geo server fails to determine an address because Google Maps services are unavailable | All     |

See additional information [here](#errors).


## JSON Requirements


### Property Names Must Be in ASCII `snake_case`

Although there is no established industry standard, our services must adhere to `snake_case` to ensure uniformity.


### Server Response Must Begin with a JSON Object

The server's response body must always return a JSON object as the top-level data structure.

This allows for compatible extensibility by adding new attributes.

Bad:

```json
[{}]
```

Good:

```json
{
	"collection": [],
	"meta": "meta information"
}
```


### Empty Array Should Not Be `null`

The server should represent an empty array with `[]`, not `null`.


### Date and Time Properties Should Have an `_at` Suffix

Properties containing date and time must end with `_at`. This distinguishes them from `true/false` values that could have identical names:

```text
created_at
modified_at
occurred_at
returned_at
```


### Date and Time Format Requirements

Use the date and time formats defined in RFC 3339, in UTC.

Bad:


```json
{
  "date_time": 1460062925000
}
```

Good:

```json
{
  "date_time": "2015-05-28T14:07:17Z"
}
```


## Specify Units of Measurement

Bad:

```json
{
  "duration": 5000
}
```

Good:

```json
{
  "duration_ms": 5000
}
```

```json
{
  "duration": "5000ms"
}
```

```json
{
  "duration": {
    "unit": "ms",
    "value": 5000
  }
}
```

Following this rule, monetary values must always include the currency code (as per ISO 4217 standards).


### Group Fields Related to the Same Domain

At every level of API interaction, entities should be logically grouped under one name where possible. Such an API is more convenient to understand than a simple listing of various attributes.

Bad:

```json
{
  "free_wifi_id": 1,
  "wifi_auth_type": "sms",
  "place_name": "McDonalds",
  "place_location_latitude": 34.4343,
  "place_location_longitude": 43.444,
  "place_open_now": true,
  "walking_distance_minutes": 10
}
```

Good:

```json
{
  "place": {
    "name": "McDonalds",
    "location": {
      "latitude": 34.4343,
      "longitude": 43.444
    },
    "open_now": true
  },
  "wifi": {
    "id": 1,
    "type": "sms"
  },
  "route": {
    "distance_km": 10,
    "duration": "10 minutes"
  }
}
```

In this case, readability is achieved through decomposition. A developer studying the interface reasons as follows: "Okay, here is the description of the Wi-Fi characteristics, which I'm not interested in at the moment because I'm currently working with the route to the establishment". If all properties are mixed into one response, unnecessary fields come into focus, significantly degrading the API design.


## Prefer Cursor Pagination

Pagination can be implemented based on offset or based on cursors.

Cursor-based pagination (prev/next links) should be preferred when designing APIs. For more information, see 
[here](https://slack.engineering/evolving-api-pagination-at-slack/).

```json
{
  "orders": [],
  "cursors": {
    "prev": "...",
    "next": "dXNlcjpXMDdRQ1JQQTQ="
  }
}
```


## Confidential Information in HTTP Requests

REST APIs should be designed with the need to prevent potential credential leaks in mind. It should be noted that web servers universally log paths and parameters used in requests.

Passwords, session tokens, and API keys in GET requests must be passed in `HTTP` headers; in `POST`, `PATCH`, or `PUT` requests, they should be passed in the body or in HTTP headers.

Avoid sending parameters via a query string, since they can end up in logs:

```text
GET https://example.com/controller/123/action?apiKey=a53f435643de32 
```

Good:

```text
GET /controller/123/action HTTP/1.1 
Authorization: a53f435643de32 
```

## Template Query Parameters

When providing sorting and pagination functionality in requests, the following naming conventions should be adhered to:

1. `sort`: a comma-separated list of values that sets the sorting order, (+) for ascending, (-) for descending, e.g., `/orders?sort=-created_at,+amount`.
2. `offset`: a numeric offset from the first element presented in the collection for offset pagination.
3. `limit`: a limit on the number of records returned in the collection. The maximum value should be restricted on the server side.
4. `cursor`: a page identifier of the last element, used for cursor-based pagination.


## Requests Initiating Asynchronous Work

There are situations where a client needs to initiate an asynchronous task on the server, and due to the nature of the task's duration, it is not possible to get an immediate response to the request.  

In such cases, the server should return a status code 202 (Accepted) in response to the request, along with a reference to the appropriate resource for the client to monitor the task status and an approximate time estimate.

```text
POST /v1/res
202 Accepted
Location: /v1/res/jobs/123
```

The resource being monitored should return a `200` status code, along with the current status of the task and an approximate time estimate.

```text
GET /v1/res/jobs/123
200 OK

{
  "state": "LOOKING_FOR_NEAR_DRIVER",
  "message": "Looking for near driver",
  "created_at": "...",
  "estimate": "...",
  "warnings": []
 }
```


## Search

Simple search in collection should be designed as "resourceful API".

```text
/users?name="Immanuel"
```

Global search should be designed as a separate endpoint:

```text
/search?q=black+smith
```

If search query returns nothing, the server should return a `200` code and an empty list inside a JSON object.
    
```json
{
    "items": []
}
```

Server shouldn't return a `404` code, as it would mean the absence of the corresponding resource, not an empty list as a result of applying a filter.


## Localization

Client should pass the appropriate tag (e.g., `"en-DE"`) in accordance with BCP 47 in the `Accept-Language` header.


## Authentication

Authentication is based on a JWT token (Bearer Authentication) `Authorization: Bearer <token>`.

In case of the absence of a token in the `Authorization` header, the server should return a `401` code. If token is expired or invalid, the server also should return a `401` code.


## HTTP Methods Usage


### `GET`

`GET` requests should be used to read a resource or a collection of resources.

1. When processing a `GET` request on an individual resource, the server returns `200` in case of success and 404 in case of absence of the resource.
2. When processing a `GET` request on a collection of resources, the server returns `200` in case of success and 404 in case of absence of the collection as an accessible resource.
3. In cases specified by business requirements, for large collections, pagination, sorting, searching, and filtering should be provided.


### `PUT`

`PUT` should be used to fully replace a resource.


### `POST`

`POST` should be used for creating a collection of resources and creating an individual resource in the collection.

In response to such a request, the server generates a resource ID and returns its representation along with the generated ID. The server most often returns a 201 code in case of success in creating a resource, but it can also return a `202` code in case of partial creation of a resource with the possibility of monitoring the long (asynchronous) work of the server.
    
When none of the methods semantically fits the required action, `POST` can be used on a residual basis. The idempotency of the method is ensured by the client passing the generated `idempotency_key` (`UUID`) in the request body.


### `PATCH`

`PATCH` is used for partial updating of resource properties. The idempotency of the method in the case of concurrent updates is ensured by the server using the `etag` field (which indicates the version of the resource) of the corresponding structure, which the client passes in the `If-Match` header. When the version differs from the one on the client, the server returns a `412` code (`precondition failed`). In case of success, a `200` code is returned.

```text
< GET /orders

> HTTP/1.1 200 OK
> {
>   "items": [
>     { "id": "O0000042", "etag": "osjnfkjbnkq3jlnksjnvkjlsbf", "foo": 42, "bar": true },
>     { "id": "O0000043", "etag": "kjshdfknjqlowjdsljdnfkjbkn", "foo": 24, "bar": false }
>   ]
> }

< PATCH /orders/O0000042
< If-Match: osjnfkjbnkq3jlnksjnvkjlsew
< { "id": "O0000042", "foo": 43, "bar": true }

> HTTP/1.1 412 Precondition failed
```


### `DELETE`

`DELETE` is used to delete a resource from a collection. Multiple filters can be used in the request parameters, for example:

```text
DELETE /bids?param1=value1&param2=value2
```

Server returns 204 in case of successful deletion of the resource and `404` if the resource was not found or was already deleted.

There are cases where it is preferable to use the `PATCH` method to change the status (e.g., order cancellation).

`DELETE` is used in case of final and irreversible deletion of a resource when there is complete confidence that there is no need to re-access the resource.


### `HEAD`

`HEAD` is similar to the `GET` method, except that the server returns only headers. This method is used when the client needs to find information about a resource without receiving its representation.


### `OPTIONS`

`OPTIONS` is used to determine the supported methods of a specific resource. The server should include the `Allow` header with a list of supported methods in the response.


# API for Mobile Devices

Endpoints that serve mobile applications should be designed with the following considerations in mind.


## Errors

For 4xx and 5xx errors, an additional structure should be returned for client error handling. The minimum required:

```json
{
  "meta": {
    "code": 400,
    "message": "Unknown error"
  }
}
```

The structure contains a numeric error code corresponding to the `HTTP` code and a useful error message. The structure could be extended this way:

* `code` - HTTP status code to simplify access to it for clients;
* `message` - a localized error message that is visible to the user;
* `title` - a localized error message title that is visible to the user;
* `action` - what actions should be taken on the client side when handling an error:
  * `view_type` - how to display the error message (dialog, toast, none);
  * `show_retry` - whether to show a retry button;
  * `button` - a custom button:
     * `text` - button text;
     * `url` - a URL to navigate to when the button is clicked. Could be a deeplink.
* `debug_id` - a unique identifier that can be used to identify the cause of the error in the logs;
* `details` - error details if multiple errors occurred on different request fields:
    * `message` - a detailed error message;
    * `fields` - the fields to which the error applies.

```json
{
  "meta": {
    "code": 403,
    "reason": "neworder_no_money",
    "message": "Not enough money on the account",
    "title": "Top up your account",
    "action": {
      "view_type": "dialog",
      "show_retry": true,
      "button": {
        "text": "Top up wallet",
        "url": "indriver://open/contractor/city/payment"
      }
    },
    "debug_id": "7673868d-231e-490d-9c4f-19288e7e668d",
    "details": [
      {
        "message": "account has no 3000 USD",
        "fields": ["price"],
      }, {
        "message": "Driver can't take this order",
        "fields": ["order_id", "contractor_id"],
      },
    ],
    }
 }
```


### Recommended Error Handling Flow

All errors should be properly handled. If `meta.action` is present, the client should follow the instructions. Some teams could build their own error handling flow based on the `reason` field. Be ready that response body could be in format.


### Recommended Actions Based on HTTP Status Code

`400 Bad Request` — notify user. Error details will be in `meta`. In the test environment, the Details property will be filled.

`401 Unauthorized` — expired or invalid JWT-token. Update or get a new JWT-token from the auth service.

`403 Forbidden` — user is not allowed to perform the operation. Notify user. Error details will be in `meta`. In the test environment, the Details property will be filled.

`404 Not Found` — object not found. Process `meta`.

`405 Method Not Allowed` — the request is made to the wrong place. For example, `POST` instead of `GET`. Show a system error.

`409 Conflict` — duplicate request with the same idempotency key. When handling the error, the action specified in `meta`, request active entities and make redirection.

`410 Gone` — this endpoint is no longer supported. The client should contact the router to get a new endpoint.

`413 Payload Too Large` — too much data was sent. Notify user. Most likely, a file that is too large was sent.

`417 Expectation Failed` — incorrect signature in the `X-Sign` header. The error occurs if the request is made by a bot. Show a system error. Such an error should not occur in a normal situation.

`419 Authentication Timeout` — incorrect or expired `X-Track-Id`. Signing important requests. Get a new `X-track-id`.

`426 Upgrade required` — to make the request, the client needs to update the client version. Notify user. Error details should be in `meta`.

`427 Antifraud` — the action is blocked by the `antifraud` service. Notify user with two buttons: the one that came in `meta` and cancel button that returns to the previous action. Error details and button 1 will be in `meta`.

`429 Too Many Requests` — the server is overloaded with similar requests. Message from rate-limiter. Retry the request according `Retry-After` header. If possible, show the client the results of the previous request. For example, you can redraw old drivers on the map based on the old response.

`467 Logical error` — the reason field must be filled in. For example, `BID_EXPIRED`. The client should handle the error according to its logic based on the reason.

`500 Internal Server Error` — internal server error. Retry the request after some time.

`502 Bad Gateway` — the server is not working. Retry the request after some time. If the server does not respond within
a reasonable time, request a new endpoint from the router.

`503 Service Unavailable` — the service is unavailable. Retry the request after some time. If the server does not respond within a reasonable time, request a new endpoint from the router.

`504 Gateway Timeout` — the request took too long to process and was dropped by the server. Retry the request after some time. If the server does not respond within a reasonable time, request a new endpoint from the router.

`521 Server is down` — the server is unavailable, technical work is being carried out. Notify the user. Error details may be in `meta`. Otherwise, show a stub.

`523 Origin Is Unreachable` — an external service related to the request is unavailable. Retry the request after some time. If possible, show the client the results of the previous request.