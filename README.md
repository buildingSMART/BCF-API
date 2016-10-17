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
  * [1.4 Cross Origin Resource Sharing (CORS)](#14-cross-origin-resource-sharing-cors)
  * [1.5 Http Status Codes](#15-http-status-codes)
  * [1.6 Error Response Body Format](#16-error-response-body-format)
  * [1.7 DateTime Format](#17-datetime-format)
  * [1.8 Authorization](#18-authorization)
    + [1.8.1 Per-Entity Authorization](#181-per-entity-authorization)
    + [1.8.2 Determining Authorized Entity Actions](#182-determining-authorized-entity-actions)
  * [1.9 Additional Response Object Properties](#19-additional-response-object-properties)
- [2. Topologies](#2-topologies)
  * [2.1 Topology 1 - BCF-Server only](#21-topology-1---bcf-server-only)
  * [2.2 Topology 2 - Colocated BCF-Server and Model Server](#22-topology-2---colocated-bcf-server-and-model-server)
- [3. Public Services](#3-public-services)
  * [3.1 Version Service](#31-version-service)
  * [3.2 Authentication Services](#32-authentication-services)
    + [3.2.1 Obtaining Authentication Information](#321-obtaining-authentication-information)
    + [3.2.2 OAuth2 Protocol Flow - Client Request](#322-oauth2-protocol-flow---client-request)
    + [3.2.3 OAuth2 Protocol Flow - Token Request](#323-oauth2-protocol-flow---token-request)
    + [3.2.4 OAuth2 Protocol Flow - Refresh Token Request](#324-oauth2-protocol-flow---refresh-token-request)
    + [3.2.5 OAuth2 Protocol Flow - Dynamic Client Registration](#325-oauth2-protocol-flow---dynamic-client-registration)
    + [3.2.6 OAuth2 Protocol Flow - Requesting Resources](#326-oauth2-protocol-flow---requesting-resources)
- [4. BCF Services](#4-bcf-services)
  * [4.1 Project Services](#41-project-services)
    + [4.1.1 GET Projects Service](#411-get-projects-service)
    + [4.1.2 GET Project Service](#412-get-project-service)
    + [4.1.3 PUT Project Service](#413-put-project-service)
    + [4.1.4 GET Project Extension Service](#414-get-project-extension-service)
    + [4.1.5 Expressing User Authorization Through Project Extensions](#415-expressing-user-authorization-through-project-extensions)
      - [4.1.5.1 Project](#4151-project)
      - [4.1.5.2 Topic](#4152-topic)
      - [4.1.5.3 Comment](#4153-comment)
      - [4.1.5.4 Viewpoint](#4154-viewpoint)
  * [4.2 Topic Services](#42-topic-services)
    + [4.2.1 GET Topics Service](#421-get-topics-service)
    + [4.2.2 POST Topic Service](#422-post-topic-service)
    + [4.2.3 GET Topic Service](#423-get-topic-service)
    + [4.2.4 PUT Topic Service](#424-put-topic-service)
    + [4.2.6 GET Topic BIM Snippet Service](#426-get-topic-bim-snippet-service)
    + [4.2.7 PUT Topic BIM Snippet Service](#427-put-topic-bim-snippet-service)
    + [4.2.8 Determining Allowed Topic Modifications](#428-determining-allowed-topic-modifications)
  * [4.3 File Services](#43-file-services)
    + [4.3.1 GET Files (Header) Service](#431-get-files-header-service)
    + [4.3.2 PUT Files (Header) Service](#432-put-files-header-service)
  * [4.4 Comment Services](#44-comment-services)
    + [4.4.1 GET Comments Service](#441-get-comments-service)
    + [4.4.2 POST Comment Service](#442-post-comment-service)
    + [4.4.3 GET Comment Service](#443-get-comment-service)
    + [4.4.4 PUT Comment Service](#444-put-comment-service)
    + [4.4.5 Determining allowed Comment modifications](#445-determining-allowed-comment-modifications)
  * [4.5 Viewpoint Services](#45-viewpoint-services)
    + [4.5.1 GET Viewpoints Service](#451-get-viewpoints-service)
    + [4.5.2 POST Viewpoint Service](#452-post-viewpoint-service)
    + [4.5.3 GET Viewpoint Service](#453-get-viewpoint-service)
    + [4.5.4 GET Viewpoint Snapshot Service](#454-get-viewpoint-snapshot-service)
    + [4.5.5 PUT Viewpoint Snapshot Service](#455-put-viewpoint-snapshot-service)
    + [4.5.6 GET Viewpoint Bitmap Service](#456-get-viewpoint-bitmap-service)
    + [4.5.7 PUT Viewpoint Bitmap Service](#457-put-viewpoint-bitmap-service)
    + [4.5.8 Determining allowed Viewpoint modifications](#458-determining-allowed-viewpoint-modifications)
  * [4.6 Component Services](#46-component-services)
    + [4.6.1 GET Components Service](#461-get-components-service)
    + [4.6.2 PUT Components Service](#462-put-components-service)
  * [4.7 Related Topics Services](#47-related-topics-services)
    + [4.7.1 GET Related Topics Service](#471-get-related-topics-service)
    + [4.7.2 PUT Related Topics Service](#472-put-related-topics-service)
  * [4.8 Document Reference Services](#48-document-reference-services)
    + [4.8.1 GET Document References Service](#481-get-document-references-service)
    + [4.8.2 POST Document Reference Service](#482-post-document-reference-service)
    + [4.8.3 PUT Document Reference Service](#483-put-document-reference-service)
  * [4.9 Document Services](#49-document-services)
    + [4.9.1 GET Documents Service](#491-get-documents-service)
    + [4.9.2 POST Document Service](#492-post-document-service)
    + [4.9.3 GET Document Service](#493-get-document-service)
  * [4.10 Topics History Services](#410-topics-history-services)
    + [4.10.1 GET Topics History Service](#4101-get-topics-history-service)
    + [4.10.2 GET Topic History Service](#4102-get-topic-history-service)

<!-- tocstop -->

# 1. Introduction

BCF is a format for managing issues on a BIM project. The BCF-API supports the exchange of BCF issues between software applications via a [RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer) web interface, which means that data is exchanged via Url-encoded query parameters and Json bodies over the Http protocol. Every resource described in this API has a corresponding Json schema (schema version draft-03).
Url schemas in this readme are relational to the BCF base servers Url if no absolute values are provided.

For security reasons, all API Http traffic should be sent via TLS/SSL over Https connection. Clients and Servers should both enforce secure connections and disallow unsafe connections.

An example of a client implementation in C# can be found here:
[https://github.com/rvestvik/BcfApiExampleClient](https://github.com/rvestvik/BcfApiExampleClient)

## 1.1 Paging, Sorting and Filtering

When requesting collections of items, the BCF-API should offer URL parameters according to the OData v4 specification. It can be found at [http://www.odata.org/documentation/](http://www.odata.org/documentation/).

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

## 1.4 Cross Origin Resource Sharing (CORS)

To work with browser based API clients using [Cross Origin Resource Sharing (Cors)](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing), servers will put the `Access-Control-Allow'` headers in the response headers and specifiy who can access the servers Json resources. The client can look for these values and proceed with accessing the resources.

In a CORS scenario, web clients expect the following headers:
* `Access-Control-Allow-Headers: Authorization, Content-Type, Accept` to allow the `Authorization`, `Content-Type` and `Accept` headers to be used via [XHR requests](https://en.wikipedia.org/wiki/XMLHttpRequest)
* `Access-Control-Allow-Methods: GET, POST, PUT, OPTIONS` to allow the Http methods the API needs
* `Access-Control-Allow-Origin: example.com` to allow XHR requests from the `example.com` domain to the BCF server

For example, Asp.Net applications in IIS need the following entries in their `web.config` file. `*` means the server allows any values.

    <httpProtocol>
        <customHeaders>
            <add name="Access-Control-Allow-Headers" value="Content-Type, Accept, X-Requested-With,  Authorization" />
            <add name="Access-Control-Allow-Methods" value="GET, POST, PUT, DELETE, OPTIONS" />
            <add name="Access-Control-Allow-Origin" value="*" />
        </customHeaders>
    </httpProtocol>

## 1.5 Http Status Codes

The BCF API relies on the regular Http Status Code definitions. Good sources are [Wikipedia](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) or the [HTTP/1.1 Specification](https://tools.ietf.org/html/rfc7231).

Generally, these response codes shall be used in the API:
* `200 - OK` for `GET` requests that return data or `PUT` requests that update data
* `201 - Created` for `POST` requests that create data

`POST` and `PUT` requests do usually include the created resource in the response body. Exceptions to this rule are described in the specific section for the resource.

## 1.6 Error Response Body Format

BCF-API has a specified error response body format [error.json](Schemas_draft-03/error.json).

## 1.7 DateTime Format

DateTime values in this API are supposed to be in ISO 8601 compliant `YYYY-MM-DDThh:mm:ss` format with optional time zone indicators. This is the same format as defined in the Xml `xs:dateTime` type as well as the result of JavaScripts Date.toJson() output.

For example, `2016-04-28T16:31:12.270+02:00` would represent _Thursday, April 28th, 2016, 16:31:12 (270ms) with a time zone offset of +2 hours relative to UTC._

## 1.8 Authorization

API implementors can optionally choose to restrict the actions a user is allowed to perform on the BCF entities
via the API. The global default authorizations for all entities are expressed in the project extensions schema and can
be locally overridden in the entities themselves.

### 1.8.1 Per-Entity Authorization

Whenever a user requests an update-able entity with the request parameter `includeAuthorization` equal to `true` the
server should include an `authorization` field in the entity containing any local variations from the global
authorization defaults for that entity. Using this information clients can decide whether to, for example, include an
"Edit" button in the UI displaying the entity depending on the actions permitted for the user.

### 1.8.2 Determining Authorized Entity Actions

The client can calculate the available set of actions for a particular entity by taking the project-wide defaults from
the project extensions, then replacing any keys defined in the entity's `authorization` map with the values specified
locally. The meaning of each of the authorization keys is outlined in outlined in
[4.1.5 Expressing User Authorization through Project Extensions](#415-expressing-user-authorization-through-project-extensions).

**Example Scenario (Topic)**

_In the Project Extensions_

    {
        "topic_actions": [],
        "topic_status": [
            "open",
            "closed",
            "confirmed"
        ]
    }

Indicating that by default:

* no modifications can be made to Topics
* Topics can be placed in `open`, `closed` or `confirmed` status

_In the Topic_

    {
        "authorization": {
            "topic_actions": [
                "update",
                "createComment",
                "createViewpoint"
            ],
            "topic_status": [
                "closed"
            ]
        }
    }

Indicating that for this topic, the current user can:

* update the Topic, or add comments or viewpoints
* place the Topic into `closed` status

## 1.9 Additional Response Object Properties

All API response Json objects may contain additional properties that are not covered by this specification.
This is to allow server implementations freedom to add additional functionality. Clients shall ignore those properties.

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

## 3.1 Version Service

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

    GET /bcf/version

**Example Response**

    Response Code: 200 - OK
    Body:
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

    GET /bcf/auth

**Example Response**

    Response Code: 200 - OK
    Body:
    {
        "oauth2_auth_url": "https://example.com/bcf/oauth2/auth",
        "oauth2_token_url": "https://example.com/bcf/oauth2/token",
        "oauth2_dynamic_client_reg_url": "https://example.com/bcf/oauth2/reg",
        "http_basic_supported": true
    }

### 3.2.2 OAuth2 Protocol Flow - Client Request

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

### 3.2.3 OAuth2 Protocol Flow - Token Request

[token_GET.json](Schemas_draft-03/Authentication/token_GET.json)

The Client uses the **"oauth2\_token_url"** to request a token. Example:

    POST https://example.com/bcf/oauth2/token

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

Alternatively all parameters may be passed in the token request body instead of using Url parameters. The expected `Content-Type` for this request is `application/x-www-form-urlencoded`.

    POST https://example.com/bcf/oauth2/token
    Body:
    grant_type=authorization_code&code=<your_access_code>&client_id=<client_id>&client_secret=<client_secret>

The access token will be returned as JSON in the response body and is an arbitrary string, guaranteed to not exceed 255 characters length.

**Example Response**

    Response Code: 201 - Created
    Body:
    {
        "access_token": "Zjk1YjYyNDQtOTgwMy0xMWU0LWIxMDAtMTIzYjkzZjc1Y2Jh",
        "token_type": "bearer",
        "expires_in": "3600",
        "refresh_token": "MTRiMjkzZTYtOTgwNC0xMWU0LWIxMDAtMTIzYjkzZjc1Y2Jh"
    }

### 3.2.4 OAuth2 Protocol Flow - Refresh Token Request

[token_GET.json](Schemas_draft-03/Authentication/token_GET.json)

The process to retrieve a refresh token is exactly the same as retrieving a token via the code workflow except the `refresh_token` is sent instead of the `code` parameter and the `refresh_token` grant type is used.

**Example Request**

    POST https://example.com/bcf/oauth2/token?grant_type=refresh_token&refresh_token=<your_refresh_token>

Alternatively all parameters may be passed in the token request body instead of using Url parameters.

    POST https://example.com/bcf/oauth2/token
    Body:
    grant_type=refresh_token&refresh_token=<your_refresh_token>&client_id=<client_id>&client_secret=<client_secret>

The access token will be returned as JSON in the response body and is an arbitrary string, guaranteed to not exceed 255 characters length.

**Example Response**

    Response Code: 201 - Created
    Body:
    {
        "access_token": "Zjk1YjYyNDQtOTgwMy0xMWU0LWIxMDAtMTIzYjkzZjc1Y2Jh",
        "token_type": "bearer",
        "expires_in": "3600",
        "refresh_token": "MTRiMjkzZTYtOTgwNC0xMWU0LWIxMDAtMTIzYjkzZjc1Y2Jh"
    }

The refresh token can only be used once to retrieve a token and a new refresh token.

### 3.2.5 OAuth2 Protocol Flow - Dynamic Client Registration

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

    Response Code: 201 - Created
    Body:
    {
        "client_id": "cGxlYXN1cmUu",
        "client_secret": "ZWFzdXJlLg=="
    }

### 3.2.6 OAuth2 Protocol Flow - Requesting Resources

When requesting other resources the access token must be passed via the `Authorization` header using the Bearer scheme (e.g. `Authorization: Bearer Zjk1YjYyNDQtOTgwMy0xMWU0LWIxMDAtMTIzYjkzZjc1Y2Jh`).

----------

# 4. BCF Services

## 4.1 Project Services

For compatibility with the project structure of existing systems, the `project_id` property of `project` resources is **not forced to be a guid but may be any string**.

### 4.1.1 GET Projects Service

**Resource URL**

    GET /bcf/{version}/projects

[project_GET.json](Schemas_draft-03/Project/project_GET.json)

Retrieve a **collection** of projects where the currently logged on user has access to.

**Example Request**

    GET /bcf/2.1/projects

**Example Response**

    Response Code: 200 - OK
    Body:
    [{
        "project_id": "F445F4F2-4D02-4B2A-B612-5E456BEF9137",
        "name": "Example project 1",
        "authorization": {
            "project_actions": [
                "createTopic",
                "createDocument"
            ]
        }
    }, {
        "project_id": "A233FBB2-3A3B-EFF4-C123-DE22ABC8414",
        "name": "Example project 2",
        "authorization": {
            "project_actions": []
        }
    }]

### 4.1.2 GET Project Service

**Resource URL**

    GET /bcf/{version}/projects/{project_id}

[project_GET.json](Schemas_draft-03/Project/project_GET.json)

Retrieve a specific project.

**Example Request**

    GET /bcf/2.1/projects/B724AAC3-5B2A-234A-D143-AE33CC18414

**Example Response**

    Response Code: 200 - OK
    Body:
    {
        "project_id": "B724AAC3-5B2A-234A-D143-AE33CC18414",
        "name": "Example project 3",
        "authorization": {
            "project_actions": [
                "update",
                "updateProjectExtensions"
            ]
        }
    }

### 4.1.3 PUT Project Service

**Resource URL**

    PUT /bcf/{version}/projects/{project_id}

[project_PUT.json](Schemas_draft-03/Project/project_PUT.json)

Modify a specific project.

**Example Request**

    PUT /bcf/2.1/projects/B724AAC3-5B2A-234A-D143-AE33CC18414
    Body:
    {
        "name": "Example project 3 - Second Section"
    }

**Example Response**

    Response Code: 200 - OK
    Body:
    {
        "project_id": "B724AAC3-5B2A-234A-D143-AE33CC18414",
        "name": "Example project 3 - Second Section",
        "authorization": {
            "project_actions": [
                "update",
                "updateProjectExtensions"
            ]
        }
    }

### 4.1.4 GET Project Extension Service

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/extensions

[extensions_GET.json](Schemas_draft-03/Project/extensions_GET.json)

Retrieve a specific projects extensions.
Project extensions are used to define possible values that can be used in topics and comments, for example topic labels and priorities. They may change during the course of a project. The most recent extensions state which values are valid at a given moment for newly created topics and comments.

**Example Request**

    GET /bcf/2.1/projects/B724AAC3-5B2A-234A-D143-AE33CC18414/extensions

**Example Response**

    Response Code: 200 - OK
    Body:
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
        ],
        "project_actions": [
            "update",
            "createTopic",
            "createDocument",
            "updateProjectExtensions"
        ],
        "topic_actions": [
            "update",
            "updateBimSnippet",
            "updateRelatedTopics",
            "updateDocumentServices",
            "updateFiles",
            "createComment",
            "createViewpoint"
        ],
        "comment_actions": [
            "update"
        ],
        "viewpoint_actions": [
            "update",
            "updateBitmap",
            "updateSnapshot",
            "updateComponent"
        ]
    }

### 4.1.5 Expressing User Authorization Through Project Extensions

Global default authorizations for the requesting user can be expressed in the project schema. The actions authorized
here will apply to any entities that do not override them locally. The complete set of options for the BCF entities are
listed below.

#### 4.1.5.1 Project

The 'project_actions' entry in the project extensions defines what actions are allowed to be performed
at the project level. The available actions include:

* *update* - The ability to update the project details (see [4.1.3 PUT Project Service](#413-put-project-service))
* *createTopic* - The ability to create a new topic (see [4.2.2 POST Topic Service](#422-post-topic-service))
* *createDocument* - The ability to create a new document (see [4.9.2 POST Document Service](#492-post-document-service))

#### 4.1.5.2 Topic

The 'topic_actions' entry in the project extensions defines what actions are allowed to be performed at the topic
level by default (i.e. unless overridden by specific topics) The available actions include:

* *update* - The ability to update the topic (see [4.2.4 PUT Topic Service](#424-put-topic-service))
* *updateBimSnippet* - The ability to update the BIM snippet for topics (see [4.2.7 PUT Topic BIM Snippet Service](#427-put-topic-bim-snippet-service))
* *updateRelatedTopics* - The ability to update the collection of related topics (see [4.7.2 PUT Related Topics Service](#472-put-related-topics-service))
* *updateDocumentReferences* - The ability to update the collection of document references (see [4.8.3 PUT Document Reference Service](#483-put-document-reference-service))
* *updateFiles* - The ability to update the file header (see [4.3.2 PUT Files (Header) Service](#432-put-files-header-service))
* *createComment* - The ability to create a comment (see [4.4.2 POST Comment Service](#442-post-comment-service))
* *createViewpoint* - The ability to create a new viewpoint (see [4.5.2 POST Viewpoint Service](#452-post-viewpoint-service))

#### 4.1.5.3 Comment

The 'comment_actions' entry in the project extensions defines what actions are allowed to be performed at the comment level by
default (i.e unless overridden by specific comments). The available actions include:

* *update* - The ability to update the comment (see [4.4.4 PUT Comment Service](#444-put-comment-service))

#### 4.1.5.4 Viewpoint

The 'viewpoint_actions' entry in the project extensions defines what actions are allowed to be performed at the viewpoint level by
default (i.e. unless overridden by specific viewpoints). The available actions include:

* *update* - The ability to update the viewpoint (see [4.5.4 PUT Viewpoint Service](#454-put-viewpoint-service))
* *updateBitmap* - The ability to update the bitmap for the viewpoint (see [4.5.8 PUT Viewpoint Bitmap Service](#458-put-viewpoint-bitmap-service))
* *updateSnapshot* - The ability to update the snapshot for the viewpoint (see [4.5.6 PUT Viewpoint Snapshot Service](#456-put-viewpoint-snapshot-service))
* *updateComponent* - The ability to update the component for the viewpoint (see [4.6.2 PUT Components Service](#462-put-components-service))

---------

## 4.2 Topic Services

### 4.2.1 GET Topics Service

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/topics

[topic_GET.json](Schemas_draft-03/Collaboration/Topic/topic_GET.json)

Retrieve a **collection** of topics related to a project (default sort order is `creation_date`).

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

Get topics that are open, assigned to Architect@example.com and created after December 5th 2015. Sort the result on last modified

    GET /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics?$filter=assigned_to eq 'Architect@example.com' and status eq 'Open' and creation_date gt datetime'2015-12-05T00:00:00+01:00'&$orderby=modified_date desc

Odata does not support list operators. To achieve list query, use the 'or' operator.
Get topics that have at least one of the labels 'Architecture', 'Structural' or 'Heating'

    GET /bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics?$filter=contains(labels, 'Architecture') or contains(labels, 'Structural') or contains(labels, 'Heating')

**Example Request**

    GET /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics

**Example Response**

    Response Code: 200 - OK
    Body:
    [{
        "guid": "A245F4F2-2C01-B43B-B612-5E456BEF8116",
        "creation_author": "Architect@example.com",
        "title": "Example topic 1",
        "labels": [
            "Architecture",
            "Structural"
        ],
        "creation_date": "2013-10-21T17:34:22.409Z"
    }, {
        "guid": "A211FCC2-3A3B-EAA4-C321-DE22ABC8414",
        "creation_author": "Architect@example.com",
        "title": "Example topic 2",
        "labels": [
            "Architecture",
            "Heating",
            "Electrical"
        ],
        "creation_date": "2014-11-19T14:24:11.316Z"
    }]

### 4.2.2 POST Topic Service

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

    POST /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics
    Body:
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

**Example Response**

    Response Code: 201 - Created
    Body:
    {
        "guid": "A245F4F2-2C01-B43B-B612-5E456BEF8116",
        "creation_author": "Architect@example.com",
        "creation_date": "2016-08-01T17:34:22.409Z",
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

### 4.2.3 GET Topic Service

**Resource URL**

    GET /bcf/{version}/projects/{guid}/topics/{guid}

[topic_GET.json](Schemas_draft-03/Collaboration/Topic/topic_GET.json)

Retrieve a specific topic.

**Example Request**

    GET /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228

**Example Response**

    Response Code: 200 - OK
    Body:
    {
        "guid": "B345F4F2-3A04-B43B-A713-5E456BEF8228",
        "creation_author": "Architect@example.com",
        "creation_date": "2016-08-01T17:34:22.409Z",
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
        },
        "authorization": {
            "topic_actions": [
                "createComment",
                "createViewpoint"
            ]
        }
    }

### 4.2.4 PUT Topic Service

**Resource URL**

    PUT /bcf/{version}/projects/{project_id}/topics/{guid}

[topic_PUT.json](Schemas_draft-03/Collaboration/Topic/topic_PUT.json)

Modify a specific topic, description similar to POST.

**Example Request**

    PUT /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228
    Body:
    {
        "topic_type": "Clash",
        "topic_status": "open",
        "title": "Example topic 3 - Changed Title",
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

**Example Response**

    Response Code: 200 - OK
    Body:
    {
        "guid": "B345F4F2-3A04-B43B-A713-5E456BEF8228",
        "creation_author": "Architect@example.com",
        "creation_date": "2016-08-01T17:34:22.409Z",
        "modified_author": "Architect@example.com",
        "modified_date": "2016-08-02T13:22:22.409Z",
        "topic_type": "Clash",
        "topic_status": "open",
        "title": "Example topic 3 - Changed Title",
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

### 4.2.6 GET Topic BIM Snippet Service

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/topics/{guid}/snippet

Retrieves a topics BIM-Snippet as binary file.

### 4.2.7 PUT Topic BIM Snippet Service

**Resource URL**

    PUT /bcf/{version}/projects/{project_id}/topics/{guid}/snippet

Puts a new BIM Snippet binary file to a topic. If this is used, the parent topics BIM Snippet property `is_external` must be set to `false` and the `reference` must be the file name with extension.

### 4.2.8 Determining Allowed Topic Modifications

The global default Topic authorizations are expressed in the project schema and when Topic(s) are requested with the
parameter "includeAuthorization" equal to "true" Topics will include an "authorization" field containing any local
overrides for each Topic.

## 4.3 File Services

### 4.3.1 GET Files (Header) Service

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/topics/{guid}/files

[file_GET.json](Schemas_draft-03/Collaboration/File/file_GET.json)

Retrieve a **collection** of file references as topic header.

**Example Request**

    GET /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/files

**Example Response**

    Response Code: 200 - OK
    Body:
    [{
        "ifc_project": "0J$yPqHBD12v72y4qF6XcD",
        "file_name": "OfficeBuilding_Architecture_0001.ifc",
        "reference": "https://example.com/files/0J$yPqHBD12v72y4qF6XcD_0001.ifc"
    }, {
        "ifc_project": "3hwBHP91jBRwPsmyf$3Hea",
        "file_name": "OfficeBuilding_Heating_0003.ifc",
        "reference": "https://example.com/files/3hwBHP91jBRwPsmyf$3Hea_0003.ifc"
    }]

### 4.3.2 PUT Files (Header) Service

**Resource URL**

    PUT /bcf/{version}/projects/{project_id}/topics/{guid}/files

[file_PUT.json](Schemas_draft-03/Collaboration/File/file_PUT.json)

Update a **collection** of file references on the topic header.

**Example Request**

    PUT /bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/files
    Body:
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

    Response Code: 200 - OK
    Body:
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

### 4.4.1 GET Comments Service

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

    GET /bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/comments?$filter=status eq 'Closed' and date gt datetime'2015-12-05T00:00:00+01:00'&$orderby=date asc

**Example Request**

    GET /bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/comments

**Example Response**

    Response Code: 200 - OK
    Body:
    [{
        "guid": "C4215F4D-AC45-A43A-D615-AA456BEF832B",
        "date": "2016-08-01T12:34:22.409Z",
        "author": "max.muster@example.com",
        "comment": "Clash found",
        "topic_guid": "B345F4F2-3A04-B43B-A713-5E456BEF8228",
        "authorization": {
            "comment_actions": [
                "update"
            ]
        }
    }, {
        "guid": "A333FCA8-1A31-CAAC-A321-BB33ABC8414",
        "date": "2016-08-01T14:24:11.316Z",
        "author": "bob.heater@example.com",
        "comment": "will rework the heating model",
        "topic_guid": "B345F4F2-3A04-B43B-A713-5E456BEF8228"
    }]

### 4.4.2 POST Comment Service

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

    POST /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/comments
    Body:
    {
        "comment": "will rework the heating model"
    }

**Example Response**

    Response Code: 201 - Created
    Body:
    {
        "guid": "A333FCA8-1A31-CAAC-A321-BB33ABC8414",
        "date": "2016-08-01T14:24:11.316Z",
        "author": "bob.heater@example.com",
        "comment": "will rework the heating model",
        "topic_guid": "B345F4F2-3A04-B43B-A713-5E456BEF8228"
    }

### 4.4.3 GET Comment Service

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/topics/{guid}/comments/{guid}

[comment_GET.json](Schemas_draft-03/Collaboration/Comment/comment_GET.json)

Get a single comment.

**Example Request**

    GET /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/comments/A333FCA8-1A31-CAAC-A321-BB33ABC8414

**Example Response**

    Response Code: 200 - OK
    Body:
    {
        "guid": "A333FCA8-1A31-CAAC-A321-BB33ABC8414",
        "date": "2016-08-01T14:24:11.316Z",
        "author": "bob.heater@example.com",
        "comment": "will rework the heating model",
        "topic_guid": "B345F4F2-3A04-B43B-A713-5E456BEF8228"
    }

### 4.4.4 PUT Comment Service

**Resource URL**

    PUT /bcf/{version}/projects/{project_id}/topics/{guid}/comments/{guid}

[comment_PUT.json](Schemas_draft-03/Collaboration/Comment/comment_PUT.json)

Update a single comment, description similar to POST.

**Example Request**

    PUT /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/comments
    Body:
    {
        "comment": "will rework the heating model and fix the ventilation"
    }

**Example Response**

    Response Code: 200 - OK
    Body:
    {
        "guid": "A333FCA8-1A31-CAAC-A321-BB33ABC8414",
        "date": "2016-08-01T14:24:11.316Z",
        "author": "bob.heater@example.com",
        "modified_date": "2016-08-01T19:24:11.316Z",
        "modified_author": "bob.heater@example.com",
        "comment": "will rework the heating model and fix the ventilation",
        "topic_guid": "B345F4F2-3A04-B43B-A713-5E456BEF8228"
    }

### 4.4.5 Determining allowed Comment modifications

The global default Comment authorizations are expressed in the project schema and when Comment(s) are requested with the
parameter "includeAuthorization" equal to "true" Comments will include an "authorization" field containing any local
overrides for each Comment.

## 4.5 Viewpoint Services

### 4.5.1 GET Viewpoints Service

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/topics/{guid}/viewpoints

[viewpoint_GET.json](Schemas_draft-03/Collaboration/Viewpoint/viewpoint_GET.json)

Retrieve a **collection** of all viewpoints related to a topic.

**Example Request**

    GET /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/viewpoints

**Example Response**

    Response Code: 200 - OK
    Body:
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
        },
        "authorization": {
            "viewpoint_actions": [
                "update",
                "updateBitmap",
                "updateSnapshot",
                "updateComponent"
            ]
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
        },
        "authorization": {
            "viewpoint_actions": []
        }
    }]

### 4.5.2 POST Viewpoint Service

**Resource URL**

    POST /bcf/{version}/projects/{project_id}/topics/{guid}/viewpoints

[viewpoint_POST.json](Schemas_draft-03/Collaboration/Viewpoint/viewpoint_POST.json)

Add a new viewpoint.

**Parameters**

JSON encoded body using the "application/json" content type.

|parameter|type|description|required|
|---------|----|-----------|--------|
| x, y, z | number | numbers defining either a point or a vector | optional |
| index | number | parameter for sorting | optional |
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

    POST /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/viewpoints
    Body:
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

    Response Code: 201 - Created
    Body:
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

### 4.5.3 GET Viewpoint Service

**Resource URL**

    GET /bcf/{version}/projects/{guid}/topics/{guid}/viewpoints/{guid}

[viewpoint_GET.json](Schemas_draft-03/Collaboration/Viewpoint/viewpoint_GET.json)

Retrieve a specific viewpoint.

**Example Request**

    GET /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/viewpoints/a11a82e7-e66c-34b4-ada1-5846abf39133

**Example Response**

    Response Code: 200 - OK
    Body:
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
        },
        "authorization": {
            "viewpoint_actions": [
                "update",
                "updateBitmap",
                "updateSnapshot",
                "updateComponent"
            ]
        }
    }

### 4.5.4 GET Viewpoint Snapshot Service

**Resource URL**

    GET /bcf/{version}/projects/{guid}/topics/{guid}/viewpoints/{guid}/snapshot

Retrieve a viewpoints snapshot (png, jpg or bmp) as image file.

**Example Request**

    GET /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/viewpoints/a11a82e7-e66c-34b4-ada1-5846abf39133/snapshot

### 4.5.5 PUT Viewpoint Snapshot Service

**Resource URL**

    PUT /bcf/{version}/projects/{guid}/topics/{guid}/viewpoints/{guid}/snapshot

Add or update a viewpoints snapshot (png, jpg or bmp).

**Example Request**

    PUT /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/viewpoints/a11a82e7-e66c-34b4-ada1-5846abf39133/snapshot

PUT Body contains binary image data

**Example Response**

    Response Code: 200 - OK
    Empty Body

### 4.5.6 GET Viewpoint Bitmap Service

**Resource URL**

    GET /bcf/{version}/projects/{guid}/topics/{guid}/viewpoints/{guid}/bitmaps/{guid}

Retrieve a specific viewpoints bitmap image file (png, jpg or bmp).

**Example Request**
    GET /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/viewpoints/a11a82e7-e66c-34b4-ada1-5846abf39133/bitmaps/760bc4ca-fb9c-467f-884f-5ecffeca8cae

### 4.5.7 PUT Viewpoint Bitmap Service

**Resource URL**

    PUT /bcf/{version}/projects/{guid}/topics/{guid}/viewpoints/{guid}/bitmaps/{guid}

Add or update a specific bitmap in a viewpoint (png, jpg or bmp).

**Example Request**

    PUT /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/viewpoints/a11a82e7-e66c-34b4-ada1-5846abf39133/bitmaps/760bc4ca-fb9c-467f-884f-5ecffeca8cae

PUT Body contains binary image data

**Example Response**

    Response Code: 200 - OK
    Empty Body

### 4.5.8 Determining allowed Viewpoint modifications

The global default Viewpoint authorizations are expressed in the project schema and when Viewpoint(s) are requested with the
parameter "includeAuthorization" equal to "true" Viewpoints will include an "authorization" field containing any local
overrides for each Viewpoint.

## 4.6 Component Services

### 4.6.1 GET Components Service

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/topics/{guid}/viewpoints/{guid}/components

[component_GET.json](Schemas_draft-03/Collaboration/Component/component_GET.json)

Retrieve a **collection** of all components related to a viewpoint.

**Example Request**

    GET /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/viewpoints/a11a82e7-e66c-34b4-ada1-5846abf39133/components

**Example Response**

    Response Code: 200 - OK
    Body:
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

### 4.6.2 PUT Components Service

**Resource URL**

    PUT /bcf/{version}/projects/{project_id}/topics/{guid}/viewpoints/{guid}/components

[component_PUT.json](Schemas_draft-03/Collaboration/Component/component_PUT.json)

Add or update a **collection** of all components related to a viewpoint.

**Example Request**

    PUT /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/viewpoints/a11a82e7-e66c-34b4-ada1-5846abf39133/components
    Body:
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

    Response Code: 200 - OK
    Body:
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

### 4.7.1 GET Related Topics Service

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/topics/{guid}/related_topics

[related_topic_GET.json](Schemas_draft-03/Collaboration/RelatedTopic/related_topic_GET.json)

Retrieve a **collection** of all related topics to a topic.

**Example Request**

    GET /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/related_topics

**Example Response**

    Response Code: 200 - OK
    Body:
    [{
        "related_topic_guid": "db49df2b-0e42-473b-a3ee-f7b785d783c4"
    }, {
        "related_topic_guid": "6963a846-54d1-4050-954d-607cd5e48aa3"
    }]

### 4.7.2 PUT Related Topics Service

**Resource URL**

    PUT /bcf/{version}/projects/{project_id}/topics/{guid}/related_topics

[related_topic_PUT.json](Schemas_draft-03/Collaboration/RelatedTopic/related_topic_PUT.json)

Add or update a **collection** of all related topics to a topic.

**Example Request**

    PUT /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/related_topics
    Body:
    [{
        "related_topic_guid": "db49df2b-0e42-473b-a3ee-f7b785d783c4"
    }, {
        "related_topic_guid": "6963a846-54d1-4050-954d-607cd5e48aa3"
    }, {
        "related_topic_guid": "bac66ab4-331e-4f21-a28e-083d2cf2e796"
    }]

**Example Response**

    Response Code: 200 - OK
    Body:
    [{
        "related_topic_guid": "db49df2b-0e42-473b-a3ee-f7b785d783c4"
    }, {
        "related_topic_guid": "6963a846-54d1-4050-954d-607cd5e48aa3"
    }, {
        "related_topic_guid": "bac66ab4-331e-4f21-a28e-083d2cf2e796"
    }]

## 4.8 Document Reference Services

### 4.8.1 GET Document References Service

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/topics/{guid}/document_references

[document_reference_GET.json](Schemas_draft-03/Collaboration/DocumentReference/document_reference_GET.json)

Retrieve a **collection** of all document references to a topic.

**Example Request**

    GET /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/document_references

**Example Response**

    Response Code: 200 - OK
    Body:
    [{
        "guid": "472ab37a-6122-448e-86fc-86503183b520",
        "referenced_document": "http://example.com/files/LegalRequirements.pdf",
        "description": "The legal requirements for buildings."
    }, {
        "guid": "6cbfe31d-95c3-4f4d-92a6-420c23698721",
        "referenced_document": "http://example.com/files/DesignParameters.pdf",
        "description": "The building owners global design parameters for buildings."
    }]

### 4.8.2 POST Document Reference Service

**Resource URL**

    POST /bcf/{version}/projects/{project_id}/topics/{guid}/document_references

[document_reference_POST.json](Schemas_draft-03/Collaboration/DocumentReference/document_reference_POST.json)

Add or update document references to a topic.

**Example Request**

    POST /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/document_references
    Body:
    [{
        "referenced_document": "http://example.com/files/LegalRequirements.pdf",
        "description": "The legal requirements for buildings."
    }]

**Example Response**

    Response Code: 201 - Created
    Body:
    [{
        "guid": "472ab37a-6122-448e-86fc-86503183b520",
        "referenced_document": "http://example.com/files/LegalRequirements.pdf",
        "description": "The legal requirements for buildings."
    }]

### 4.8.3 PUT Document Reference Service

**Resource URL**

    PUT /bcf/{version}/projects/{project_id}/topics/{guid}/document_references/{guid}

[document_reference_PUT.json](Schemas_draft-03/Collaboration/DocumentReference/document_reference_PUT.json)

Add or update document references to a topic.

**Example Request**

    PUT /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/document_references/472ab37a-6122-448e-86fc-86503183b520
    Body:
    [{
        "guid": "472ab37a-6122-448e-86fc-86503183b520",
        "referenced_document": "http://example.com/files/LegalRequirements_Update.pdf",
        "description": "The legal requirements for buildings."
    }]

**Example Response**

    Response Code: 200 - OK
    Body:
    [{
        "guid": "472ab37a-6122-448e-86fc-86503183b520",
        "referenced_document": "http://example.com/files/LegalRequirements_Update.pdf",
        "description": "The legal requirements for buildings."
    }]

## 4.9 Document Services

### 4.9.1 GET Documents Service

[document_GET.json](Schemas_draft-03/Collaboration/Document/document_GET.json)

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/documents

Retrieve a **collection** of all documents uploaded to a project.

**Example Request**

    GET /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/documents

**Example Response**

    Response Code: 200 - OK
    Body:
    [{
        "guid": "472ab37a-6122-448e-86fc-86503183b520",
        "filename": "LegalRequirements.pdf"
    }, {
        "guid": "6cbfe31d-95c3-4f4d-92a6-420c23698721",
        "filename": "DesignParameters.pdf"
    }]

### 4.9.2 POST Document Service

**Resource URL**

    POST /bcf/{version}/projects/{project_id}/documents

Upload a document (binary file) to a project.

**Example Request**

    POST /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/documents

**Example Response**

    Response Code: 201 - Created
    Body:
    {
        "guid": "472ab37a-6122-448e-86fc-86503183b520",
        "filename": "Official_Building_Permission.pdf"
    }

### 4.9.3 GET Document Service

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/documents/{guid}

Retrieves a document as binary file.

**Example Request**

    GET /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/documents/472ab37a-6122-448e-86fc-86503183b520

**Example Response**

Retrieves a document as binary file.

## 4.10 Topics History Services

The topic history service reflects the history for topics. Each creation or update of a topic generates a new topic history.

### 4.10.1 GET Topics History Service

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/topics/history

[topic_history_GET.json](Schemas_draft-03/Collaboration/History/topic_history_GET.json)

Retrieve a **collection** of topic histories related to a project (default sort order is `date`).

**Topic history types**

|type|operation|value|
|---------|-----------|-----------|
|topic|created|null|
|title|updated|The title (limit: 128 characters)|
|description|updated, removed|The description (limit: 1024 characters) or null if removed|
|status|updated|The status (value from extensions) |
|type|updated|The type (value from extensions)|
|priority|updated, removed|The priority (value from extensions) or null if removed|
|due_date|updated, removed|The due date or null if removed|
|assigned|updated, removed|The assigned user (value from extensions) or null if removed|
|label|added, removed|The label added or removed (value from extensions)|

**Odata filter parameters**

|parameter|type|description|
|---------|----|-----------|
|topic_guid|string|guid of the topic |
|author|string|userId of the author (value from extensions)|
|type|string|type of the history (value from Topic history types, table above)|
|date|datetime|date of the history|

**Odata sort parameters**

|parameter|description|
|---------|-----------|
|date|date of the history|

**Example Request with odata**

Get histories of type 'status_set' made by Architect@example.com and created after December 5th 2015. Sort the result on least recent

    GET /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/history?$filter=author eq 'Architect@example.com' and type eq 'status_set' and date gt datetime'2015-12-05T00:00:00+01:00'&$orderby=date asc

Get latest histories of given topic. Skip the 10 first, and get the 5 next

    GET /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/history?$filter=topic_guid eq 'A245F4F2-2C01-B43B-B612-5E456BEF8116'&$top=5&$skip=10

Odata does not support list operators. To achieve list query, use the 'or' operator.
Get histories that is of type 'status_set', 'type_set' or 'title_set'

    /bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/history?$filter=type eq 'status_set' or type eq 'type_set' or type eq 'title_set'

**Example Request**

    GET /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/history

**Example Response**

    Response Code: 200 - OK
    Body:
    [{
        "topic_guid": "A211FCC2-3A3B-EAA4-C321-DE22ABC8414",
        "date": "2014-11-19T14:24:11.316Z",
        "author": "Architect@example.com",
        "events": [
            {
                "type": "status",
                "operation": "update",
                "value": "Closed"
            }
        ]
    }, {
        "topic_guid": "A245F4F2-2C01-B43B-B612-5E456BEF8116",
        "date": "2013-10-21T17:34:22.409Z",
        "author": "Architect@example.com",
        "events": [
            {
                "type": "type",
                "operation": "update",
                "value": "Warning"
            }
        ]
    }]

### 4.10.2 GET Topic History Service

**Resource URL**

    GET /bcf/{version}/projects/{project_id}/topics/{topic_guid}/history

[topic_history_GET.json](Schemas_draft-03/Collaboration/History/topic_history_GET.json)

Retrieve a **collection** of topic histories related to a project (default sort order is `date`).

**Topic history types**

|type|operation|value|
|---------|-----------|-----------|
|topic|created|null|
|title|updated|The title (limit: 128 characters)|
|description|updated, removed|The description (limit: 1024 characters) or null if removed|
|status|updated|The status (value from extensions) |
|type|updated|The type (value from extensions)|
|priority|updated, removed|The priority (value from extensions) or null if removed|
|due_date|updated, removed|The due date or null if removed|
|assigned|updated, removed|The assigned user (value from extensions) or null if removed|
|label|added, removed|The label added or removed (value from extensions)|

**Odata filter parameters**

|parameter|type|description|
|---------|----|-----------|
|author|string|userId of the author (value from extensions)|
|type|string|type of the history (value from Topic history types, table above)|
|date|datetime|date of the history|

**Odata sort parameters**

|parameter|description|
|---------|-----------|
|date|date of the history|

**Example Request with odata**

Get histories of type 'status_set' made by Architect@example.com and created after December 5th 2015. Sort the result on least recent

    GET /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/A245F4F2-2C01-B43B-B612-5E456BEF8116/history?$filter=author eq 'Architect@example.com' and type eq 'status_set' and date gt datetime'2015-12-05T00:00:00+01:00'&$orderby=date asc

Get latest histories. Skip the 10 first, and get the 5 next

    GET /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/A245F4F2-2C01-B43B-B612-5E456BEF8116/history?$top=5&$skip=10

Odata does not support list operators. To achieve list query, use the 'or' operator.
Get histories that is of type 'status_set', 'type_set' or 'title_set'

    /bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/A245F4F2-2C01-B43B-B612-5E456BEF8116/history?$filter=type eq 'status_set' or type eq 'type_set' or type eq 'title_set'

**Example Request**

    GET /bcf/2.1/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/A245F4F2-2C01-B43B-B612-5E456BEF8116/history

**Example Response**

    Response Code: 200 - OK
    Body:
    [{
        "topic_guid": "A245F4F2-2C01-B43B-B612-5E456BEF8116",
        "date": "2014-11-19T14:24:11.316Z",
        "author": "Architect@example.com",
        "events": [
            {
                "type": "type",
                "operation": "update",
                "value": "Error"
            }
        ]
    }, {
        "topic_guid": "A245F4F2-2C01-B43B-B612-5E456BEF8116",
        "date": "2013-10-21T17:34:22.409Z",
        "author": "Architect@example.com",
        "events": [
            {
                "type": "status",
                "operation": "update",
                "value": "Open"
            }
        ]
    }]
