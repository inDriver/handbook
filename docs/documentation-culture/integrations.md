# Integrations


## Technical Requirements

* overview (why the integration is required, short description of the integration);
* API keys;
* Libraries;
* Dependencies.


## Registration and Key Management

* registration process;
* obtaining keys;
* key generation.


## Environment Setup

* frontend configuration;
* backend configuration.


## Database Integration

Add information on database required for successful integration.


## Request Details

* URL;
* method;
* description;
* authentication;
* format;
* request parameters, their requirements, and acceptable values;
* example request (JSON).


## Possible Errors

* invalid API key: the provided API key is invalid;
* missing required parameters: some required parameters are missing;
* invalid parameter values: Some parameter values are invalid;
* unauthorized access: the user does not have permission to access the resource;
* resource not found: the requested resource could not be found;
* server error: an internal server error occurred request to try again later.


# API


## Internal API

Internal API is used only by the company's engineers.

`.yaml` and `.json` formats are used to create API and render a Swagger on the Documentary Platform.


## External API

External API is usually developed and provided for integrators.

The structure of the document shall be as following:

Handler.

Request example:

```plaintext
curl -X 'GET' \
  'vscodewebview://1qgjfnnp6748d6cmgghofac6c7v1ouop9mtg5uko8l07bdccbm8h/api/v1/token?partner_name=abaco' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer token'
```

| Header | Value |
| --- | --- |
|  |  |

Response example:

| Parameter | Type | Mandatory or optional | Description | Comment |
| --- | --- | --- | --- | --- |

Response codes:

| Code | Description |
| --- | --- |
|  |  |


For API writing it is also recommended to use [Google developers style guide](https://developers.google.com/style/api-reference-comments).




