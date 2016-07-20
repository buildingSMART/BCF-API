# BCF REST API
![](https://raw.githubusercontent.com/BuildingSMART/BCF/master/Icons/BCFicon128.png)

**Version 2.1** based on BCFv2.1.
[GitHub repository](https://github.com/BuildingSMART/BCF)

**Table of Contents**

<!-- toc -->

- [1. Introduction](#1-introduction)
  * [1.1 Paging, Sorting and Filtering](#11-paging-sorting-and-filtering)
  * [1.2 Caching](#12-caching)
  * [1.3 Updating Resources via HTTP PUT](#13-updating-resources-via-http-put)
  * [1.4 Cross origin resource sharing (Cors)](#14-cross-origin-resource-sharing-cors)
  * [1.5 HTTP status codes](#15-http-status-codes)
  * [1.6 Error response body format](#16-error-response-body-format)
  * [1.7 DateTime format](#17-datetime-format)
- [2. Topologies](#2-topologies)
  * [2.1 Topology 1 - BCF-Server only](#21-topology-1---bcf-server-only)
  * [2.2 Topology 2 - Colocated BCF-Server and Model Server](#22-topology-2---colocated-bcf-server-and-model-server)
- [3. Public Services](#3-public-services)
  * [3.1 Information Services](#31-information-services)
  * [3.2 Authentication Services](#32-authentication-services)
    + [3.2.1 Obtaining Authentication Information](#321-obtaining-authentication-information)
    + [3.2.2 OAuth2 protocol flow - Client Request -](#322-oauth2-protocol-flow---client-request--)
    + [3.2.3 OAuth2 protocol flow - Token Request -](#323-oauth2-protocol-flow---token-request--)
    + [3.2.4 OAuth2 protocol flow - Refresh Token Request -](#324-oauth2-protocol-flow---refresh-token-request--)
    + [3.2.5 OAuth2 protocol flow - Dynamic Client Registration -](#325-oauth2-protocol-flow---dynamic-client-registration--)
    + [3.2.6 OAuth2 protocol flow - Requesting Resources -](#326-oauth2-protocol-flow---requesting-resources--)
- [4. BCF Services](#4-bcf-services)
  * [4.1 Project Services](#41-project-services)
    + [4.1.1 GET Project Services](#411-get-project-services)
    + [4.1.2 GET Single Project Services](#412-get-single-project-services)
    + [4.1.3 PUT Single Project Services](#413-put-single-project-services)
    + [4.1.4 GET Project Extension Services](#414-get-project-extension-services)
    + [4.1.5 POST Project Extension Services](#415-post-project-extension-services)
    + [4.1.6 PUT Project Extension Services](#416-put-project-extension-services)
  * [4.2 Topic Services](#42-topic-services)
    + [4.2.1 GET Topic Services](#421-get-topic-services)
    + [4.2.2 POST Topic Services](#422-post-topic-services)
    + [4.2.3 GET Single Topic Services](#423-get-single-topic-services)
    + [4.2.4 PUT Single Topic Services](#424-put-single-topic-services)
    + [4.2.6 GET Topic BIM Snippet](#426-get-topic-bim-snippet)
    + [4.2.7 PUT Topic BIM Snippet](#427-put-topic-bim-snippet)
  * [4.3 File Services](#43-file-services)
    + [4.3.1 GET File (Header) Services](#431-get-file-header-services)
    + [4.3.2 PUT File (Header) Services](#432-put-file-header-services)
  * [4.4 Comment Services](#44-comment-services)
    + [4.4.1 GET Comment Services](#441-get-comment-services)
    + [4.4.2 POST Comment Services](#442-post-comment-services)
    + [4.4.3 GET Single Comment Services](#443-get-single-comment-services)
    + [4.4.4 PUT Single Comment Services](#444-put-single-comment-services)
  * [4.5 Viewpoint Services](#45-viewpoint-services)
    + [4.5.1 GET Viewpoint Services](#451-get-viewpoint-services)
    + [4.5.2 POST Viewpoint Services](#452-post-viewpoint-services)
    + [4.5.3 GET Single Viewpoint Services](#453-get-single-viewpoint-services)
    + [4.5.4 PUT Single Viewpoint Services](#454-put-single-viewpoint-services)
    + [4.5.5 GET snapshot of a Viewpoint Service](#455-get-snapshot-of-a-viewpoint-service)
    + [4.5.6 PUT snapshot of a Viewpoint Service](#456-put-snapshot-of-a-viewpoint-service)
    + [4.5.7 GET bitmap of a Viewpoint Service](#457-get-bitmap-of-a-viewpoint-service)
    + [4.5.8 PUT bitmap of a Viewpoint Service](#458-put-bitmap-of-a-viewpoint-service)
  * [4.6 Component Services](#46-component-services)
    + [4.6.1 GET Component Services](#461-get-component-services)
    + [4.6.2 PUT Component Services](#462-put-component-services)
  * [4.7 Related Topics Services](#47-related-topics-services)
    + [4.7.1 GET Related Topics Services](#471-get-related-topics-services)
    + [4.7.2 PUT Related Topics Services](#472-put-related-topics-services)
  * [4.8 Document Reference Services](#48-document-reference-services)
    + [4.8.1 GET Document Reference Services](#481-get-document-reference-services)
    + [4.8.2 PUT Document Reference Services](#482-put-document-reference-services)
  * [4.9 Document Services](#49-document-services)
    + [4.9.1 GET Documents Services](#491-get-documents-services)
    + [4.9.2 POST Document Services](#492-post-document-services)
    + [4.9.3 GET Document Services](#493-get-document-services)

<!-- tocstop -->

# 1. Introduction

BCF is a format for managing issues on a BIM project. RESTful BCF-API supports the exchange of BCFv2 issues between software applications.

All API access is transmitted over HTTPS. Data is sent as URL encoded query parameters and JSON POST bodies and received as JSON. Every resource has a corresponding JSON Schema (Draft 03). JSON Hyper Schema is used for link definition. The authentication method is OAuth2. URL schemas in this readme are relational to the base server URL.

An example of a client implementation in C# can be found here:
[https://github.com/rvestvik/BcfApiExampleClient](https://github.com/rvestvik/BcfApiExampleClient)

## 1.1 Paging, Sorting and Filtering

When requesting collections of items, the BCF-API should offer URL parameters according to the OData specification. It can be found at [http://www.odata.org/documentation/](http://www.odata.org/documentation/).

## 1.2 Caching

ETags, or entity-tags, are an important part of HTTP, being a critical part of caching, and also used in "conditional" requests.

The ETag response-header field value, an entity tag, provides for an "opaque" cache validator.
The easiest way to think of an etag is as an MD5 or SHA1 hash of all the bytes in a representation. If just one byte in the representation changes, the etag will change.

ETags are returned in a response to a GET:

    joe@joe-laptop:~$ curl --include http://bitworking.org/news/
    HTTP/1.1 200 Ok
    Date: Wed, 21 Mar 2007 15:06:15 GMT
    Server: Apache
    ETag: "078de59b16c27119c670e63fa53e5b51"
    Content-Length: 23081
    …

The client may send an "If-None-Match" HTTP Header containing the last retrieved etag. If the content has not changed the server returns a status code 304 (not modified) and no response body.

## 1.3 Updating Resources via HTTP PUT

Whenever a resource offers the HTTP PUT method to be updated as a whole.

This means that there is no partial update mechanism for objects but every PUT request is sending the whole object representation. PUT schemas may exclude server generated values that cannot be edited, such as creation dates or authors.

## 1.4 Cross origin resource sharing (Cors)

The server will put the "Access-Control-Allow-Headers" in the response header and specify who can access the servers (JSON) resources. The client can look for this value and proceed with accessing the resources.

The server has a web config file .. "*" means the server allow the resources for all domains.

    <httpProtocol>
        <customHeaders>
            <add name="Access-Control-Allow-Headers" value="Content-Type, Accept, X-Requested-With,  Authorization" />
            <add name="Access-Control-Allow-Methods" value="GET, POST, PUT, DELETE, OPTIONS" />
            <add name="Access-Control-Allow-Origin" value="*" />
        </customHeaders>
    </httpProtocol>

## 1.5 HTTP status codes

The BCF API relies on the regular Http Status Code definitions. Good sources are [Wikipedia](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) or the [HTTP/1.1 Specification](https://tools.ietf.org/html/rfc7231).

## 1.6 Error response body format

BCF-API has a specified error response body format [error.json](Schemas_draft-03/error.json).

## 1.7 DateTime format

DateTime values in this API are supposed to be in ISO 8601 compliant `YYYY-MM-DDThh:mm:ss` format with optional time zone indicators. This is the same format as defined in the Xml `xs:dateTime` type as well as the result of JavaScripts Date.toJson() output.

For example, `2016-04-28-16:31.27+2:00` would represent _Thursday, April 28th, 2016, 16:31 (270ms) with a time zone offset of +2 hours relative to UTC._

----------

# 2. Topologies

## 2.1 Topology 1 - BCF-Server only

Model collaboration is managed through a shared file server or a network file sharing service like Dropbox. The BCF-Server handles the authentication and the BCF-Issues.

![Topology1](Images/Topology1.png)

## 2.2 Topology 2 - Colocated BCF-Server and Model Server

BCF and model server are co-located on the same hosts.

![Topology3](Images/Topology3.png)

----------
# 3. Public Services

## 3.1 Information Services

[version_GET.json](Schemas_draft-03/Public/version_GET.json)

**Resource URL (public resource)**

    GET /bcf/version

**Parameters**

|Parameter|Type|Description|Required|
|---------|----|-----------|--------|
|version|string|Identifier of the version|true|
|detailed_version|string|Url to specification on GitHub|false|

Returns a list of all supported BCF API versions of the server.

**Example Request**

    GET https://example.com/bcf/version

**Example Response**

    {
        "versions": [{
            "version_id": "1.0",
            "detailed_version": "https://github.com/BuildingSMART/BCF-API"
        }, {
            "version_id": "2.1",
            "detailed_version": "https://github.com/BuildingSMART/BCF-API"
        }]
    }

----------

## 3.2 Authentication Services

### 3.2.1 Obtaining Authentication Information

[auth_GET.json](Schemas_draft-03/Authentication/auth_GET.json)

Authentication is based on the [OAuth 2.0 Protocol](http://tools.ietf.org/html/draft-ietf-oauth-v2-22).

**Resource URL (public resource)**

    GET /bcf/auth

**Parameters**

|Parameter|Type|Description|Required|
|---------|----|-----------|--------|
|oauth2_auth_url|string|URL to authorisation page|true|
|oauth2_token_url|string|URL for token requests|true|
|oauth2_dynamic_client_reg_url|string|URL for automated client registration|true|
|http_basic_supported|boolean|Indicates if Http Basic Authentication is supported|true|

**Example Request**

    GET https://example.com/bcf/auth

**Example Response**

    {
        "oauth2_auth_url": "https://example.com/bcf/oauth2/auth",
        "oauth2_token_url": "https://example.com/bcf/oauth2/token",
        "oauth2_dynamic_client_reg_url": "https://example.com/bcf/oauth2/reg",
        "http_basic_supported": true
    }

### 3.2.2 OAuth2 protocol flow - Client Request -

The Client uses the **"oauth2\_auth_url"** and adds the following parameters to it.

|parameter|value|
|-------------|------|
|response_type|`code` as string literal|
|client_id|your client id|
|state|unique user defined value|

Example URL:

    GET https://example.com/bcf/oauth2/auth?response_type=code&client_id=<your_client_id>&state=<user_defined_string>

Example redirected URL:

    https://YourWebsite.com/retrieveCode?code=<server_generated_code>&state=<user_defined_string>

Tip:
You can use the state parameter to transport custom information.

**Open a browser window or redirect the user to this resource.** This redirects back to the specified redirect URI with the provided state and the authorization code as a query parameter if the user allows your app to access the account, the value "access_denied" in the error query parameter if the user denies access.

### 3.2.3 OAuth2 protocol flow - Token Request -

[token_GET.json](Schemas_draft-03/Authentication/token_GET.json)

The Client uses the **"oauth2\_token_url"** to request a token. Example:

    POST https://example.com/bcf/oauth2/token
    Content type: application/x-www-form-urlencoded.

**Parameters**

|parameter|type|description|
|---------|----|-----------|
|access_token|string|The issued OAuth2 token|
|token_type|string|Always `bearer`|
|expires_in|integer|The lifetime of the access token in seconds|
|refresh_token|string|The issued OAuth2 refresh token, one-time-usable only|

The POST request can be done via HTTP Basic Authorization with your applications `client_id` as the username and your `client_secret` as the password.

**Example Request**

    POST https://example.com/bcf/oauth2/token?grant_type=authorization_code&code=<your_access_code>
    Headers:
    Authorization: Basic <client_id:client_secret>

Alternatively all parameters may be passed in the token request body instead of using Url parameters.

    POST https://example.com/bcf/oauth2/token
    Body:
    grant_type=authorization_code&code=<your_access_code>&client_id=<client_id>&client_secret=<client_secret>

The access token will be returned as JSON in the response body and is an arbitrary string, guaranteed to not exceed 255 characters length.

**Example Response**

    {
        "access_token": "Zjk1YjYyNDQtOTgwMy0xMWU0LWIxMDAtMTIzYjkzZjc1Y2Jh",
        "token_type": "bearer",
        "expires_in": "3600",
        "refresh_token": "MTRiMjkzZTYtOTgwNC0xMWU0LWIxMDAtMTIzYjkzZjc1Y2Jh"
    }

### 3.2.4 OAuth2 protocol flow - Refresh Token Request -

[token_GET.json](Schemas_draft-03/Authentication/token_GET.json)

The process to retrieve a refresh token is exactly the same as retrieving a token via the code workflow except the `refresh_token` is sent instead of the `code` parameter and the `refresh_token` grant type is used.

**Example Request**

    POST https://example.com/bcf/oauth2/token?grant_type=refresh_token&refresh_token=<your_refresh_token>
    Headers:
    Authorization: Basic <client_id:client_secret>

Alternatively all parameters may be passed in the token request body instead of using Url parameters.

    POST https://example.com/bcf/oauth2/token
    Body:
    grant_type=refresh_token&refresh_token=<your_refresh_token>&client_id=<client_id>&client_secret=<client_secret>

The access token will be returned as JSON in the response body and is an arbitrary string, guaranteed to not exceed 255 characters length.

**Example Response**

    {
        "access_token": "Zjk1YjYyNDQtOTgwMy0xMWU0LWIxMDAtMTIzYjkzZjc1Y2Jh",
        "token_type": "bearer",
        "expires_in": "3600",
        "refresh_token": "MTRiMjkzZTYtOTgwNC0xMWU0LWIxMDAtMTIzYjkzZjc1Y2Jh"
    }

The refresh token can only be used once to retrieve a token and a new refresh token.

### 3.2.5 OAuth2 protocol flow - Dynamic Client Registration -

[dynRegClient\_POST.json](Schemas_draft-03/Authentication/dynRegClient_POST.json)

[dynRegClient\_GET.json](Schemas_draft-03/Authentication/dynRegClient_GET.json)

The following part describes the optional dynamic registration process of a client. BCF-Servers may offer additional processes registering clients, for example allowing a client application developer to register his client on the servers website.

The resource Url for this service is server specific and is returned as `oauth2_dynamic_client_reg_url` in the `GET /bcf/auth` resource.

Register a new client :

**Parameters**

JSON encoded body using the "application/json" content type.

|parameter|type|description|
|---------|----|-----------|
|client_name|string (max. length 60)|The client name|
|client_description|string (max. length 4000)|The client description|
|client_url|string|An URL providing additional information about the client|
|redirect_url|string|An URL where users are redirected after granting access to the client|

**Example Request**

    POST https://example.com/bcf/oauth2/reg
    Body:
    {
        "client_name": "Example Application",
        "client_description": "Example CAD desktop application",
        "client_url": "http://example.com",
        "redirect_url": "http://localhost:8080"
    }

**Example Response**

    {
        "client_id": "cGxlYXN1cmUu",
        "client_secret": "ZWFzdXJlLg=="
    }

### 3.2.6 OAuth2 protocol flow - Requesting Resources -

When requesting other resources the access token must be passed via the `Authorization` header using the Bearer scheme (e.g. `Authorization: Bearer Zjk1YjYyNDQtOTgwMy0xMWU0LWIxMDAtMTIzYjkzZjc1Y2Jh`).

----------

# 4. BCF Services

## 4.1 Project Services

### 4.1.1 GET Project Services

**Resource URL**

    GET /bcf/{version}/projects

[project_GET.json](Schemas_draft-03/Project/project_GET.json)

Retrieve a **collection** of projects where the currently logged on user has access to.

**Example Request**

    GET https://example.com/bcf/2.1/projects

**Example Response**

    [{
        "project_id": "F445F4F2-4D02-4B2A-B612-5E456BEF9137",
        "name": "Example project 1"
    }, {
        "project_id": "A233FBB2-3A3B-EFF4-C123-DE22ABC8414",
        "name": "Example project 2"
    }]

### 4.1.2 GET Single Project Services

**Resource URL**

    GET /bcf/{version}/projects/{project_id}

[project_GET.json](Schemas_draft-03/Project/project_GET.json)

Retrieve a specific project.

**Example Request**

    GET https://example.com/bcf/2.1/projects/B724AAC3-5B2A-234A-D143-AE33CC18414

**Example Response**

    {
        "project_id": "B724AAC3-5B2A-234A-D143-AE33CC18414",
        "name": "Example project 3"
    }

### 4.1.3 PUT Single Project Services

**Resource URL**

    PUT /bcf/{version}/projects/{project_id}

[project_PUT.json](Schemas_draft-03/Project/project_PUT.json)

Modify a specific project.

**Example Request**

    PUT https://example.com/bcf/2.1/projects/B724AAC3-5B2A-234A-D143-AE33CC18414
    Body:
    {
        "name": "Example project 3 - Second Section"
    }

**Example Response**

    {
        "project_id": "B724AAC3-5B2A-234A-D143-AE33CC18414",
        "name": "Example project 3 - Second Section"
    }

### 4.1.4 GET Project Extension Services

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/extensions

[extensions_GET.json](Schemas_draft-03/Project/extensions_GET.json)

Retrieve a specific projects extensions.
Project extensions are used to define possible values that can be used in topics and comments, for example topic labels and priorities. They may change during the course of a project. The most recent extensions state which values are valid at a given moment for newly created topics and comments.

**Example Request**

    https://example.com/bcf/1.0/projects/B724AAC3-5B2A-234A-D143-AE33CC18414/extensions

**Example Response**

    {
        "topic_type": [
            "Information",
            "Error"
        ],
        "topic_status": [
            "Open",
            "Closed",
            "ReOpened"
        ],
        "topic_label": [
            "Architecture",
            "Structural",
            "MEP"
        ],
        "snippet_type": [
            ".ifc",
            ".csv"
        ],
        "priority": [
            "Low",
            "Medium",
            "High"
        ],
        "user_id_type": [
            "Architect@example.com",
            "BIM-Manager@example.com",
            "bob_heater@example.com"
        ]
    }

---------

## 4.2 Topic Services

### 4.2.1 GET Topic Services

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/topics

[topic_GET.json](Schemas_draft-03/Collaboration/Topic/topic_GET.json)

Retrieve a **collection** of topics related to a project (default sort order is creation_date).

**Odata filter parameters**

|parameter|type|description|
|---------|----|-----------|
|creation_author|string|userId of the creation author (value from extensions)|
|modified_author|string|userId of the modified author (value from extensions)|
|assigned_to|string|userId of the assigned person (value from extensions)|
|topic_status|string|status of a topic (value from extensions)|
|topic_type|string|type of a topic (value from extensions)|
|creation_date|datetime|creation date of a topic|
|modified_date|datetime|modification date of a topic|
|labels|array (string)|labels of a topic (value from extensions)|

**Odata sort parameters**

|parameter|description|
|---------|-----------|
|creation_date|creation date of a topic|
|modified_date|modification date of a topic|
|index|index of a topic|

**Example Request with odata**

Get topics that are open, assigned to Architect@example.com and created after December 5 2015. Sort the result on last modified

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics?$filter=assigned_to eq 'Architect@example.com' and status eq 'Open' and creation_date gt datetime'2015-12-05T00:00:00+01:00'&$orderby=modified_date desc

Odata does not support list operators. To achieve list query, use the 'or' operator.
Get topics that have at least one of the labels 'Architecture', 'Structural' or 'Heating'

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics?$filter=contains(labels, 'Architecture') or contains(labels, 'Structural') or contains(labels, 'Heating')

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics

**Example Response**

    [{
        "guid": "A245F4F2-2C01-B43B-B612-5E456BEF8116",
        "title": "Example topic 1",
        "labels": [
            "Architecture",
            "Structural"
        ],
        "creation_date": "2013-10-21T17:34:22.409Z"
    }, {
        "guid": "A211FCC2-3A3B-EAA4-C321-DE22ABC8414",
        "title": "Example topic 2",
        "labels": [
            "Architecture",
            "Heating",
            "Electrical"
        ],
        "creation_date": "2014-11-19T14:24:11.316Z"
    }]

### 4.2.2 POST Topic Services

**Resource URL**

    POST /bcf/{version}/projects/{project_id}/topics

[topic_POST.json](Schemas_draft-03/Collaboration/Topic/topic_POST.json)

Add a new topic.

**Parameters**

JSON encoded body using the "application/json" content type.

|Parameter|Type|Description|Required|
|---------|----|-----------|--------|
|topic_type|string|The type of a topic (value from extension.xsd)|false|
|topic_status|string|The status of a topic (value from extension.xsd)|false|
|reference_links|array (string)|Reference links, i.e. links to referenced resources|false|
|title|string|The title of a topic|true|
|priority|string|The priority of a topic (value from extension.xsd)|false|
|index|integer|The index of a topic|false|
|labels|array (string)|The collection of labels of a topic (values from extension.xsd)|false|
|assigned_to|string|UserID assigned to a topic (value from extension.xsd)|false|
|description|string|Description of a topic|false|
|bim_snippet.snippet_type|string|Type of a BIM-Snippet of a topic (value from extension.xsd)|false|
|bim_snippet.is_external|boolean|Is the BIM-Snippet external (default = false)|false|
|bim_snippet.reference|string|Reference of a BIM-Snippet of a topic|false|
|bim_snippet.reference_schema|string|Schema of a BIM-Snippet of a topic|false|
|due_date|string|Until when the topics issue needs to be resolved|false|

_Note: If "bim_snippet" is present, then all four properties (`snippet_type`, `is_external`, `reference` and `reference_schema`) are mandatory._

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics

    POST Body:
    {
        "topic_type": "Clash",
        "topic_status": "open",
        "title": "Example topic 3",
        "priority": "high",
        "labels": [
            "Architecture",
            "Heating"
        ],
        "assigned_to": "harry.muster@example.com",
        "bim_snippet": {
            "snippet_type": "clash",
            "is_external": true,
            "reference": "https://example.com/bcf/1.0/ADFE23AA11BCFF444122BB",
            "reference_schema": "https://example.com/bcf/1.0/clash.xsd"
        }
    }

### 4.2.3 GET Single Topic Services

**Resource URL**

    GET /bcf/{version}/projects/{guid}/topics/{guid}

[topic_GET.json](Schemas_draft-03/Collaboration/Topic/topic_GET.json)

Retrieve a specific topic.

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228

**Example Response**

    {
        "guid": "B345F4F2-3A04-B43B-A713-5E456BEF8228",
        "topic_type": "Clash",
        "topic_status": "open",
        "title": "Example topic 3",
        "priority": "high",
        "labels": [
            "Architecture",
            "Heating"
        ],
        "assigned_to": "harry.muster@example.com",
        "bim_snippet": {
            "snippet_type": "clash",
            "is_external": true,
            "reference": "https://example.com/bcf/1.0/ADFE23AA11BCFF444122BB",
            "reference_schema": "https://example.com/bcf/1.0/clash.xsd"
        }
    }

### 4.2.4 PUT Single Topic Services

**Resource URL**

    PUT /bcf/{version}/projects/{project_id}/topics/{guid}

[topic_PUT.json](Schemas_draft-03/Collaboration/Topic/topic_PUT.json)

Modify a specific topic, description similar to POST.

### 4.2.6 GET Topic BIM Snippet

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/topics/{guid}/snippet

Retrieves a topics BIM-Snippet as binary file. Will use the following HTTP headers to deliver additional information:

    Content-Type: application/octet-stream;
    Content-Length: {Size of file in bytes};
    Content-Disposition: attachment; filename="{Filename.extension}";

### 4.2.7 PUT Topic BIM Snippet

**Resource URL**

    PUT /bcf/{version}/projects/{project_id}/topics/{guid}/snippet

Puts a new BIM Snippet binary file to a topic. If this is used, the parent topics BIM Snippet property must be set to "is_external"=false and the "reference" must be the file name with extension. The following HTTP headers are used for the upload:

    Content-Type: application/octet-stream;
    Content-Length: {Size of file in bytes};

## 4.3 File Services

### 4.3.1 GET File (Header) Services

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/topics/{guid}/files

[file_GET.json](Schemas_draft-03/Collaboration/File/file_GET.json)

Retrieve a **collection** of file references as topic header.

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/files

**Example Response**

    [{
        "ifc_project": "0J$yPqHBD12v72y4qF6XcD",
        "file_name": "OfficeBuilding_Architecture_0001.ifc",
        "reference": "https://example.com/files/0J$yPqHBD12v72y4qF6XcD_0001.ifc"
    }, {
        "ifc_project": "3hwBHP91jBRwPsmyf$3Hea",
        "file_name": "OfficeBuilding_Heating_0003.ifc",
        "reference": "https://example.com/files/3hwBHP91jBRwPsmyf$3Hea_0003.ifc"
    }]

### 4.3.2 PUT File (Header) Services

**Resource URL**

    PUT /bcf/{version}/projects/{project_id}/topics/{guid}/files

[file_PUT.json](Schemas_draft-03/Collaboration/File/file_PUT.json)

Update a **collection** of file references on the topic header.

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/files

    [{
        "ifc_project": "0J$yPqHBD12v72y4qF6XcD",
        "file_name": "OfficeBuilding_Architecture_0001.ifc",
        "reference": "https://example.com/files/0J$yPqHBD12v72y4qF6XcD_0001.ifc"
    }, {
        "ifc_project": "3hwBHP91jBRwPsmyf$3Hea",
        "file_name": "OfficeBuilding_Heating_0003.ifc",
        "reference": "https://example.com/files/3hwBHP91jBRwPsmyf$3Hea_0003.ifc"
    }]

**Example Response**

    [{
        "ifc_project": "0J$yPqHBD12v72y4qF6XcD",
        "file_name": "OfficeBuilding_Architecture_0001.ifc",
        "reference": "https://example.com/files/0J$yPqHBD12v72y4qF6XcD_0001.ifc"
    }, {
        "ifc_project": "3hwBHP91jBRwPsmyf$3Hea",
        "file_name": "OfficeBuilding_Heating_0003.ifc",
        "reference": "https://example.com/files/3hwBHP91jBRwPsmyf$3Hea_0003.ifc"
    }]

## 4.4 Comment Services

### 4.4.1 GET Comment Services

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/topics/{guid}/comments

[comment_GET.json](Schemas_draft-03/Collaboration/Comment/comment_GET.json)

Retrieve a **collection** of all comments related to a topic (default ordering is date).

**Odata filter parameters**

|parameter|type|description|
|---------|----|-----------|
|author|string|userId of the author (value from extensions)|
|status|string|status of a comment (value from extensions)|
|date|datetime|creation date of a comment|

**Odata sort parameters**

|parameter|description|
|---------|-----------|
|date|creation date of a comment|

**Example Request with odata**

Get comments that are closed and created after December 5 2015. Sort the result on first created

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/comments?$filter=status eq 'Closed' and date gt datetime'2015-12-05T00:00:00+01:00'&$orderby=date asc

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/comments

**Example Response**

    [{
        "guid": "C4215F4D-AC45-A43A-D615-AA456BEF832B",
        "date": "2013-10-21T17:34:22.409Z",
        "author": "max.muster@example.com",
        "comment": "Clash found",
        "topic_guid": "B345F4F2-3A04-B43B-A713-5E456BEF8228"
    }, {
        "guid": "A333FCA8-1A31-CAAC-A321-BB33ABC8414",
        "date": "2013-11-19T14:24:11.316Z",
        "author": "bob.heater@example.com",
        "comment": "will rework the heating model",
        "topic_guid": "B345F4F2-3A04-B43B-A713-5E456BEF8228"
    }]

### 4.4.2 POST Comment Services

**Resource URL**

    POST /bcf/{version}/projects/{project_id}/topics/{guid}/comments

[comment_POST.json](Schemas_draft-03/Collaboration/Comment/comment_POST.json)

Add a new comment to a topic.

**Parameters**

JSON encoded body using the "application/json" content type.

|Parameter|Type|Description|Required|
|---------|----|-----------|--------|
|comment|string|The comment text|true|
|viewpoint_guid|string|The GUID of the related viewpoint|false|
|reply_to_comment_guid|string|GUID of the comment to which this comment replies to|false|

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/comments

    {
        "comment": "will rework the heating model"
    }

**Example Response**

    {
        "guid": "A333FCA8-1A31-CAAC-A321-BB33ABC8414",
        "verbal_status": "closed",
        "status": "closed",
        "date": "2013-11-19T14:24:11.316Z",
        "author": "bob.heater@example.com",
        "comment": "will rework the heating model",
        "topic_guid": "B345F4F2-3A04-B43B-A713-5E456BEF8228"
    }

### 4.4.3 GET Single Comment Services

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/topics/{guid}/comments/{guid}

[comment_GET.json](Schemas_draft-03/Collaboration/Comment/comment_GET.json)

Get a single comment.

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/comments/A333FCA8-1A31-CAAC-A321-BB33ABC8414

**Example Response**

    {
        "guid": "A333FCA8-1A31-CAAC-A321-BB33ABC8414",
        "date": "2013-11-19T14:24:11.316Z",
        "author": "bob.heater@example.com",
        "comment": "will rework the heating model",
        "topic_guid": "B345F4F2-3A04-B43B-A713-5E456BEF8228"
    }

### 4.4.4 PUT Single Comment Services

**Resource URL**

    PUT /bcf/{version}/projects/{project_id}/topics/{guid}/comments/{guid}

[comment_PUT.json](Schemas_draft-03/Collaboration/Comment/comment_PUT.json)

Update a single comment, description similar to POST.

## 4.5 Viewpoint Services

### 4.5.1 GET Viewpoint Services

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/topics/{guid}/viewpoints

[viewpoint_GET.json](Schemas_draft-03/Collaboration/Viewpoint/viewpoint_GET.json)

Retrieve a **collection** of all viewpoints related to a topic.

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/viewpoints

**Example Response**

    [{
        "guid": "b24a82e9-f67b-43b8-bda0-4946abf39624",
        "perspective_camera": {
            "camera_view_point": {
                "x": 0.0,
                "y": 0.0,
                "z": 0.0
            },
            "camera_direction": {
                "x": 1.0,
                "y": 1.0,
                "z": 2.0
            },
            "camera_up_vector": {
                "x": 0.0,
                "y": 0.0,
                "z": 1.0
            },
            "field_of_view": 90.0
        },
        "lines": {
            "line": [{
                "start_point": {
                    "x": 2.0,
                    "y": 1.0,
                    "z": 1.0
                },
                "end_point": {
                    "x": 0.0,
                    "y": 1.0,
                    "z": 0.7
                }
            }]
        },
        "clipping_planes": {
            "clipping_plane": [{
                "location": {
                    "x": 0.7,
                    "y": 0.3,
                    "z": -0.2
                },
                "direction": {
                    "x": 1.0,
                    "y": 0.4,
                    "z": 0.1
                }
            }]
        }
    }, {
        "guid": "a11a82e7-e66c-34b4-ada1-5846abf39133",
        "perspective_camera": {
            "camera_view_point": {
                "x": 0.0,
                "y": 0.0,
                "z": 0.0
            },
            "camera_direction": {
                "x": 1.0,
                "y": 1.0,
                "z": 2.0
            },
            "camera_up_vector": {
                "x": 0.0,
                "y": 0.0,
                "z": 1.0
            },
            "field_of_view": 90.0
        },
        "lines": {
            "line": [{
                "start_point": {
                    "x": 1.0,
                    "y": 1.0,
                    "z": 1.0
                },
                "end_point": {
                    "x": 0.0,
                    "y": 0.0,
                    "z": 0.0
                }
            }]
        },
        "clipping_planes": {
            "clipping_plane": [{
                "location": {
                    "x": 0.5,
                    "y": 0.5,
                    "z": 0.5
                },
                "direction": {
                    "x": 1.0,
                    "y": 0.0,
                    "z": 0.0
                }
            }]
        }
    }]
### 4.5.2 POST Viewpoint Services

**Resource URL**

    POST /bcf/{version}/projects/{project_id}/topics/{guid}/viewpoints

[viewpoint_POST.json](Schemas_draft-03/Collaboration/Viewpoint/viewpoint_POST.json)

Add a new viewpoint.

**Parameters**

JSON encoded body using the "application/json" content type.

|parameter|type|description|required|
|---------|----|-----------|--------|
| x, y, z | number | numbers defining either a point or a vector | optional |
| orthogonal camera | object | orthogonal camera view | optional |
| camera_view_point | object | viewpoint of the camera | optional |
| camera_directiont | object | direction of the camera | optional |
| camera_up_vector | object | direction of camera up | optional |
| view_to_world_scale | object | proportion of camera view to model | optional |
| perspective camera | object | perspective view of the camera | optional |
| camera_view_point | object | viewpoint of the camera | optional |
| camera_directiont | object | direction of the camera | optional |
| camera_up_vector | object | direction of camera up | optional |
| field_of_view | object | field of view | optional |
| line | object | graphical line | optional |
| start_point | object | start point of the line | optional |
| end_point | object | end point of the line | optional |
| clipping_plane | object | clipping plane for the model view | optional |
| location | object | origin of the clipping plane | optional |
| direction | object | direction of the clipping plane | optional |
| bitmaps | array | array of embedded pictures in the viewpoint | optional |
| guid | string | guid for the bitmap | mandatory |
| bitmap_type | enum | format of the bitmap. Predefined values `png` = 0, `jpg` = 1, `bmp` = 2 | mandatory |
| location | object | location of the center of the bitmap in world coordinates (point) | optional |
| normal | object | normal vector of the bitmap (vector) | optional |
| up | object | up vector of the bitmap (vector) | optional |

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/viewpoints
    {
        "perspective_camera": {
            "camera_view_point": {
                "x": 0.0,
                "y": 0.0,
                "z": 0.0
            },
            "camera_direction": {
                "x": 1.0,
                "y": 1.0,
                "z": 2.0
            },
            "camera_up_vector": {
                "x": 0.0,
                "y": 0.0,
                "z": 1.0
            },
            "field_of_view": 90.0
        },
        "lines": {
            "line": [{
                "start_point": {
                    "x": 1.0,
                    "y": 1.0,
                    "z": 1.0
                },
                "end_point": {
                    "x": 0.0,
                    "y": 0.0,
                    "z": 0.0
                }
            }]
        },
        "clipping_planes": {
            "clipping_plane": [{
                "location": {
                    "x": 0.5,
                    "y": 0.5,
                    "z": 0.5
                },
                "direction": {
                    "x": 1.0,
                    "y": 0.0,
                    "z": 0.0
                }
            }]
        }
    }

**Example Response**

    {
        "guid": "a11a82e7-e66c-34b4-ada1-5846abf39133",
        "perspective_camera": {
            "camera_view_point": {
                "x": 0.0,
                "y": 0.0,
                "z": 0.0
            },
            "camera_direction": {
                "x": 1.0,
                "y": 1.0,
                "z": 2.0
            },
            "camera_up_vector": {
                "x": 0.0,
                "y": 0.0,
                "z": 1.0
            },
            "field_of_view": 90.0
        },
        "lines": {
            "line": [{
                "start_point": {
                    "x": 1.0,
                    "y": 1.0,
                    "z": 1.0
                },
                "end_point": {
                    "x": 0.0,
                    "y": 0.0,
                    "z": 0.0
                }
            }]
        },
        "clipping_planes": {
            "clipping_plane": [{
                "location": {
                    "x": 0.5,
                    "y": 0.5,
                    "z": 0.5
                },
                "direction": {
                    "x": 1.0,
                    "y": 0.0,
                    "z": 0.0
                }
            }]
        }
    }

### 4.5.3 GET Single Viewpoint Services

**Resource URL**

    GET /bcf/{version}/projects/{guid}/topics/{guid}/viewpoints/{guid}

[viewpoint_GET.json](Schemas_draft-03/Collaboration/Viewpoint/viewpoint_GET.json)

Retrieve a specific viewpoint.

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/viewpoints/a11a82e7-e66c-34b4-ada1-5846abf39133

**Example Response**

    {
        "guid": "a11a82e7-e66c-34b4-ada1-5846abf39133",
        "perspective_camera": {
            "camera_view_point": {
                "x": 0.0,
                "y": 0.0,
                "z": 0.0
            },
            "camera_direction": {
                "x": 1.0,
                "y": 1.0,
                "z": 2.0
            },
            "camera_up_vector": {
                "x": 0.0,
                "y": 0.0,
                "z": 1.0
            },
            "field_of_view": 90.0
        },
        "lines": {
            "line": [{
                "start_point": {
                    "x": 1.0,
                    "y": 1.0,
                    "z": 1.0
                },
                "end_point": {
                    "x": 0.0,
                    "y": 0.0,
                    "z": 0.0
                }
            }]
        },
        "clipping_planes": {
            "clipping_plane": [{
                "location": {
                    "x": 0.5,
                    "y": 0.5,
                    "z": 0.5
                },
                "direction": {
                    "x": 1.0,
                    "y": 0.0,
                    "z": 0.0
                }
            }]
        }
    }

### 4.5.4 PUT Single Viewpoint Services

**Resource URL**

    PUT /bcf/{version}/projects/{guid}/topics/{guid}/viewpoints/{guid}

[viewpoint_PUT.json](Schemas_draft-03/Collaboration/Viewpoint/viewpoint_PUT.json)

Update a single viewpoint, description similar to POST.

### 4.5.5 GET snapshot of a Viewpoint Service

**Resource URL**

    GET /bcf/{version}/projects/{guid}/topics/{guid}/viewpoints/{guid}/snapshot

Retrieve a viewpoints snapshot (png, jpg or bmp).

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/viewpoints/a11a82e7-e66c-34b4-ada1-5846abf39133/snapshot

**Example Response**

HTTP-response header:

    Content-Type: image/png

### 4.5.6 PUT snapshot of a Viewpoint Service

**Resource URL**

    PUT /bcf/{version}/projects/{guid}/topics/{guid}/viewpoints/{guid}/snapshot

Add or update a viewpoints snapshot (png, jpg or bmp).

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/viewpoints/a11a82e7-e66c-34b4-ada1-5846abf39133/snapshot

HTTP PUT request header:

    Content-Type: image/png

PUT Body contains binary image data

**Example Response**

HTTP-response status code:

201 created (empty response body)

### 4.5.7 GET bitmap of a Viewpoint Service

**Resource URL**

    GET /bcf/{version}/projects/{guid}/topics/{guid}/viewpoints/{guid}/bitmaps/{guid}

Retrieve a specific viewpoint bitmaps image file (png, jpg or bmp).

**Example Request**
    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/viewpoints/a11a82e7-e66c-34b4-ada1-5846abf39133/bitmaps/760bc4ca-fb9c-467f-884f-5ecffeca8cae

**Example Response**

HTTP-response header:

    Content-Type: image/png

### 4.5.8 PUT bitmap of a Viewpoint Service

**Resource URL**

    PUT /bcf/{version}/projects/{guid}/topics/{guid}/viewpoints/{guid}/bitmaps/{guid}

Add or update a specific bitmap in a viewpoint (png, jpg or bmp).

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/viewpoints/a11a82e7-e66c-34b4-ada1-5846abf39133/bitmaps/760bc4ca-fb9c-467f-884f-5ecffeca8cae

HTTP-PUT request header:

    Content-Type: image/png

PUT Body contains binary image data

**Example Response**

HTTP-response status code:

201 created (empty response body)

## 4.6 Component Services

### 4.6.1 GET Component Services

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/topics/{guid}/viewpoints/{guid}/components

[component_GET.json](Schemas_draft-03/Collaboration/Component/component_GET.json)

Retrieve a **collection** of all components related to a viewpoint.

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/viewpoints/a11a82e7-e66c-34b4-ada1-5846abf39133/components

**Example Response**

    [{
        "ifc_guid": "2MF28NhmDBiRVyFakgdbCT",
        "selected": true,
        "visible": true,
        "color": "0A00FF",
        "originating_system": "Example CAD Application",
        "authoring_tool_id": "EXCAD/v1.0"
    }, {
        "ifc_guid": "3$cshxZO9AJBebsni$z9Yk",
        "selected": true,
        "visible": true,
        "color": "0A00FF",
        "originating_system": "Example CAD Application",
        "authoring_tool_id": "EXCAD/v1.0"
    }]

### 4.6.2 PUT Component Services

**Resource URL**

    POST /bcf/{version}/projects/{project_id}/topics/{guid}/viewpoints/{guid}/components

[component_PUT.json](Schemas_draft-03/Collaboration/Component/component_PUT.json)

Add or update a **collection** of all components related to a viewpoint.

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/viewpoints/a11a82e7-e66c-34b4-ada1-5846abf39133/components

    [{
        "ifc_guid": "2MF28NhmDBiRVyFakgdbCT",
        "selected": true,
        "visible": true,
        "color": "0A00FF",
        "originating_system": "Example CAD Application",
        "authoring_tool_id": "EXCAD/v1.0"
    }, {
        "ifc_guid": "3$cshxZO9AJBebsni$z9Yk",
        "selected": true,
        "visible": true,
        "color": "0A00FF",
        "originating_system": "Example CAD Application",
        "authoring_tool_id": "EXCAD/v1.0"
    }]

**Example Response**

    [{
        "ifc_guid": "2MF28NhmDBiRVyFakgdbCT",
        "selected": true,
        "visible": true,
        "color": "0A00FF",
        "originating_system": "Example CAD Application",
        "authoring_tool_id": "EXCAD/v1.0"
    }, {
        "ifc_guid": "3$cshxZO9AJBebsni$z9Yk",
        "selected": true,
        "visible": true,
        "color": "0A00FF",
        "originating_system": "Example CAD Application",
        "authoring_tool_id": "EXCAD/v1.0"
    }]

## 4.7 Related Topics Services

### 4.7.1 GET Related Topics Services

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/topics/{guid}/related_topics

[related_topic_GET.json](Schemas_draft-03/Collaboration/RelatedTopic/related_topic_GET.json)

Retrieve a **collection** of all related topics to a topic.

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/related_topics

**Example Response**

    [{
        "related_topic_guid": "db49df2b-0e42-473b-a3ee-f7b785d783c4"
    }, {
        "related_topic_guid": "6963a846-54d1-4050-954d-607cd5e48aa3"
    }]

### 4.7.2 PUT Related Topics Services

**Resource URL**

    POST /bcf/{version}/projects/{project_id}/topics/{guid}/related_topics

[related_topic_PUT.json](Schemas_draft-03/Collaboration/RelatedTopic/related_topic_PUT.json)

Add or update a **collection** of all related topics to a topic.

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/related_topics

    [{
        "related_topic_guid": "db49df2b-0e42-473b-a3ee-f7b785d783c4"
    }, {
        "related_topic_guid": "6963a846-54d1-4050-954d-607cd5e48aa3"
    }, {
        "related_topic_guid": "bac66ab4-331e-4f21-a28e-083d2cf2e796"
    }]

**Example Response**

    [{
        "related_topic_guid": "db49df2b-0e42-473b-a3ee-f7b785d783c4"
    }, {
        "related_topic_guid": "6963a846-54d1-4050-954d-607cd5e48aa3"
    }, {
        "related_topic_guid": "bac66ab4-331e-4f21-a28e-083d2cf2e796"
    }]

## 4.8 Document Reference Services

### 4.8.1 GET Document Reference Services

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/topics/{guid}/document_references

[document_reference_GET.json](Schemas_draft-03/Collaboration/DocumentReference/document_reference_GET.json)

Retrieve a **collection** of all document references to a topic.

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/document_references

**Example Response**

    [{
        "guid": "472ab37a-6122-448e-86fc-86503183b520",
        "referenced_document": "http://example.com/files/LegalRequirements.pdf",
        "description": "The legal requirements for buildings."
    }, {
        "guid": "6cbfe31d-95c3-4f4d-92a6-420c23698721",
        "referenced_document": "http://example.com/files/DesignParameters.pdf",
        "description": "The building owners global design parameters for buildings."
    }]

### 4.8.2 PUT Document Reference Services

**Resource URL**

    POST /bcf/{version}/projects/{project_id}/topics/{guid}/document_references

[document_reference_PUT.json](Schemas_draft-03/Collaboration/DocumentReference/document_reference_PUT.json)

Add or update document references to a topic.
The PUT object may either contain the property "guid" to reference a document stored on the BCF server (see section 4.9) OR the property "referenced_document" to point to an external resource.

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/document_references

    [{
        "referenced_document": "http://example.com/files/LegalRequirements.pdf",
        "description": "The legal requirements for buildings."
    }]

**Example Response**

    [{
        "guid": "472ab37a-6122-448e-86fc-86503183b520",
        "referenced_document": "http://example.com/files/LegalRequirements.pdf",
        "description": "The legal requirements for buildings."
    }]

## 4.9 Document Services

### 4.9.1 GET Documents Services

[document_GET.json](Schemas_draft-03/Collaboration/Document/document_GET.json)

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/documents

Retrieve a **collection** of all documents uploaded to a project.

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/documents

**Example Response**

    [{
        "guid": "472ab37a-6122-448e-86fc-86503183b520",
        "filename": "LegalRequirements.pdf"
    }, {
        "guid": "6cbfe31d-95c3-4f4d-92a6-420c23698721",
        "filename": "DesignParameters.pdf"
    }]

### 4.9.2 POST Document Services

**Resource URL**

    POST /bcf/{version}/projects/{project_id}/documents

Upload a document (binary file) to a project. The following HTTP headers are used for the upload:

    Content-Type: application/octet-stream;
    Content-Length: {Size of file in bytes};
    Content-Disposition: attachment; filename="{Filename.extension}";

**Example Response**

    {
        "guid": "472ab37a-6122-448e-86fc-86503183b520",
        "filename": "Official_Building_Permission.pdf"
    }

### 4.9.3 GET Document Services

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/documents/{guid}

Retrieves a document as binary file. Will use the following HTTP headers to deliver additional information:

    Content-Type: application/octet-stream;
    Content-Length: {Size of file in bytes};
    Content-Disposition: attachment; filename="{Filename.extension}";
