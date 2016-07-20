
# BCF REST API #
![](https://raw.githubusercontent.com/BuildingSMART/BCF/master/Icons/BCFicon128.png)

**Version 1.0** based on BCFv2.
[GitHub repository](https://github.com/BuildingSMART/BCF)

**Table of Contents**

[TOC]

# 1. Introduction

BCF is a format for managing issues on a BIM project. RESTful BCF-API supports the exchange of BCFv2 issues between software applications.

All API access is transmitted over HTTPS. Data is sent as URL encoded query parameters and JSON POST bodies and received as JSON. Every resource has a corresponding JSON Schema (Draft 03). JSON Hyper Schema is used for link definition. The authentication method is OAuth2. URL schemas in this readme are relational to the base server URL.



An example of a client implementation in C# can be found here:
[https://github.com/rvestvik/BcfApiExampleClient](https://github.com/rvestvik/BcfApiExampleClient)


## 1.1 Paging, Sorting and Filtering

When requesting collections of items, the BCF-API should offer URL parameters according to the OData specification. It can be found at [http://www.odata.org/documentation/](http://www.odata.org/documentation/).


## 1.2 Caching ##

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
    …..
  
The client may send an "If-None-Match" HTTP Header containing the last retrieved etag. If the content has not changed the server returns a status code 304 (not modified) and no response body.

## 1.3 Updating Resources

Whenever a resource offers the HTTP PUT method to be updated as a whole, the resource may also partially updated with a HTTP PATCH query that only transports the changed properties of the entity.


## 1.4 Cross origin resource sharing (Cors) ##

The server will put the "Access-Control-Allow-Headers" in the response header and specify who can access the servers (JSON) resources. The client can look for this value and proceed with accessing the resources. 

The server has a web config file .. "*" means the server allow the resources for all domains.

    <httpProtocol>
      <customHeaders>
    	<add name="Access-Control-Allow-Headers" value="Content-Type, Accept, X-Requested-With,  Authorization" />
    	<add name="Access-Control-Allow-Methods" value="GET, POST, PUT, DELETE, OPTIONS" />
    	<add name="Access-Control-Allow-Origin" value="*" />
      </customHeaders>
     </httpProtocol>


## 1.5 HTTP status codes ##

The BCF API relies on the regular Http Status Code definitions. Good sources are [Wikipedia](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) or the [HTTP/1.1 Specification](https://tools.ietf.org/html/rfc7231).

## 1.6 Error response body format ##

BCF-API has a specified error response body format [error.json](Schemas_draft-03/error.json).

## 1.7 DateTime format

DateTime values in this API are supposed to be in ISO 8601 compliant `YYYY-MM-DDThh:mm:ss` format with optional time zone indicators. This is the same format as defined in the Xml `xs:dateTime` type as well as the resolt of JavaScripts Date.toJson() output.

For example, `2016-04-28-16:31.27+2:00` would represent _Thursday, April 28th, 2016, 16:31 (270ms) with a time zone offset of +2 hours relative to UTC._

----------


# 2. Topologies #

## 2.1 Topology 1 - BCF-Server only##

Model collaboration is managed through a shared file server or a network file sharing service like Dropbox. The BCF-Server handles the authentication and the BCF-Issues. 

![Topology1](Images/Topology1.png)



## 2.2 Topology 2 - Colocated BCF-Server and Model Server##

BCF and model server are co-located on the same hosts.


![Topology3](Images/Topology3.png)


----------
# 3. Public Services #

## 3.1 Information Services ##

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

	https://example.com/bcf/version

**Example Response**

	{
      "versions": [
        {
	      "version_id": "1.0",
	      "detailed_version": "https://github.com/BuildingSMART/BCF-API"
        }
      ]
	}

----------


## 3.2 Authentication Services ##

### 3.2.1 Obtaining Authentication Information ###

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

    https://example.com/bcf/auth

**Example Response**

    {
      "oauth2_auth_url": "https://example.com/bcf/oauth2/auth",
      "oauth2_token_url": "https://example.com/bcf/oauth2/token",
      "oauth2_dynamic_client_reg_url": "https://example.com/bcf/oauth2/reg",
      "http_basic_supported": true
    }


### 3.2.2 OAuth2 protocol flow - Client Request -###

The Client uses the **"oauth2\_auth_url"** and adds the following parameters to it.

<table border="1">
  <tr>
    <td>response_type</td>
    <td>"code"</td>
  </tr>
  <tr>
    <td>client_id</td>
    <td>your client_id</td>
  </tr>
  <tr>
    <td>state</td>
    <td>unique user defined value</td>
  </tr>
</table>


Example URL:

	https://example.com/bcf/oauth2/auth?response_type=code&client_id=<YourClientID>&state=D98F9B4F-5B0E-4948-B8B5-59F4FE23B8E0


Example redirected URL:
	
	https://YourWebsite.com/retrieveCode?code=ABC1234567890XYZ&state=D98F9B4F-5B0E-4948-B8B5-59F4FE23B8E0

Tip:
You can use the state parameter to transport custom information. 

Open a browser window or redirect the user to this resource. This redirects back to the specified redirect URI with the provided state and the authorization code as a query parameter if the user allows your app to access the account, the value "access_denied" in the error query parameter if the user denies access.

### 3.2.3 OAuth2 protocol flow - Token Request -###

[token_GET.json](Schemas_draft-03/Authentication/token_GET.json)

The Client uses the **"oauth2\_token_url"** to request a token. Example:

	POST https://example.com/bcf/oauth2/token
	Content type: application/x-www-form-urlencoded.
 
**Parameters**
<table>
	<tr>
		<td>access_token</td>
		<td>string</td>
		<td>The issued OAuth2 token</td>
		<td>mandatory</td>
	</tr>
	<tr>
		<td>token_type</td>
		<td>string</td>
		<td>Always "bearer"</td>
		<td>mandatory</td>
	</tr>
	<tr>
		<td>expires_in</td>
		<td>integer</td>
		<td>The lifetime of the access token in seconds</td>
		<td>mandatory</td>
	</tr>
	<tr>
		<td>refresh_token</td>
		<td>string</td>
		<td>The issued OAuth2 refresh token, one-time-usable only</td>
		<td>mandatory</td>
	</tr>
</table>

The POST request can be done via HTTP Basic Authorization with your applications "ClientID" as the username and your "ClientSecret" as the password.

**Post Request Body:** 

	grant_type=authorization_code&code=<YourAccessCode>

Alternatively all parameters may be passed in the token request body

**Post Request Body:** 

	grant_type=authorization_code&code=<YourAccessCode>&client_id=<ClientID>&client_secret=<ClientSecret>

The access token will be returned as JSON in the response body and is an arbitrary string, guaranteed to fit in a varchar(255) field.

**Example Response**

	{
		"access_token":"Zjk1YjYyNDQtOTgwMy0xMWU0LWIxMDAtMTIzYjkzZjc1Y2Jh",
		"token_type":"bearer",
		"expires_in":"3600",
		"refresh_token":"MTRiMjkzZTYtOTgwNC0xMWU0LWIxMDAtMTIzYjkzZjc1Y2Jh"
	}


### 3.2.4 OAuth2 protocol flow - Refresh Token Request -###

[token_GET.json](Schemas_draft-03/Authentication/token_GET.json)


The process to retrieve a refresh token is exactly the same as retrieving a token except the POST Request Body.

POST Request Body:

	grant_type=refresh_token&refresh_token=<YourRefreshToken>

The refresh token can only be used once to retrieve a token and a new refresh token. 


### 3.2.5 OAuth2 protocol flow - Dynamic Client Registration -###

[dynRegClient\_POST.json](Schemas_draft-03/Authentication/dynRegClient_POST.json)

[dynRegClient\_GET.json](Schemas_draft-03/Authentication/dynRegClient_GET.json) 

 The following part describes the optional dynamic registration process of a client. BCF-Servers may offer additional processes registering clients, for example allowing a client application developer to register his client on the servers website.

**Resource URL**

    POST <oauth2_dynamic_client_reg_url> (obtained from auth_GET)

Register a new client :

**Parameters**

JSON encoded body using the "application/json" content type.

<table border="1">

  <tr>
    <td>client_name</td>
    <td>string (max. 60)</td>
    <td>The client name</td>
  </tr>
  <tr>
    <td>client_description</td>
    <td>string (max. 4000)</td>
    <td>The client description</td>
  </tr>
  <tr>
    <td>client_url</td>
    <td>string</td>
    <td>An URL providing additional information about the client</td>
  </tr>
  <tr>
    <td>redirect_url</td>
    <td>string</td>
    <td>An URL where users are redirected after granting access to the client</td>
  </tr>
</table>


**Example Request**

    https://example.com/bcf/oauth2/reg
	{
    "client_name": "Example Application",
	"client_description": "Example CAD desktop application",
	"client_url": "http://example.com",
	"redirect_url": "http://localhost:8080"
	}


**Example Response**


    {
      "client_id": "cGxlYXN1cmUu"
      "client_secret": "ZWFzdXJlLg==",
    }



### 3.2.6 OAuth2 protocol flow - Requesting Resources -###

When requesting other resources the access token must be passed via the Authorization header using the Bearer scheme *(e.g. Authorization: Bearer T9UNRV4sC9vr7ga)*.

----------

# 4. BCF Services #

## 4.1 Project Services ##

### 4.1.1 GET Project Services ###

**Resource URL**

    GET /bcf/{version}/projects

[project_GET.json](Schemas_draft-03/Project/project_GET.json)

Retrieve a **collection** of projects where the currently logged on user has access to.


**Example Request**

    https://example.com/bcf/1.0/projects

**Example Response**


    [{
        "project_id": "F445F4F2-4D02-4B2A-B612-5E456BEF9137",
		"name": "Example project 1"
    },
    {
        "project_id": "A233FBB2-3A3B-EFF4-C123-DE22ABC8414",
		"name": "Example project 2"
    }]



### 4.1.2 POST Project Services ###

**Resource URL**

    POST /bcf/{version}/projects

[project_POST.json](Schemas_draft-03/Project/project_POST.json)

Add a new project.

**Parameters**

JSON encoded body using the "application/json" content type.

<table border="1">

  <tr>
    <td>name</td>
    <td>string</td>
    <td>The project name</td>
  </tr>
</table>


**Example Request**

    https://example.com/bcf/1.0/projects
	{
    "name": "Example project 3"
	}

**Example Response**


    {
      "project_id": "B724AAC3-5B2A-234A-D143-AE33CC18414"
      "name": "Example project 3",
    }

### 4.1.3 GET Single Project Services ###


**Resource URL**

    GET /bcf/{version}/projects/{project_id}

[project_GET.json](Schemas_draft-03/Project/project_GET.json)

Retrieve a specific project.

**Example Request**

    https://example.com/bcf/1.0/projects/B724AAC3-5B2A-234A-D143-AE33CC18414


**Example Response**


    {
      "project_id": "B724AAC3-5B2A-234A-D143-AE33CC18414",
      "name": "Example project 3"
    }


### 4.1.4 PUT Single Project Services ###

**Resource URL**

    PUT /bcf/{version}/projects/{project_id}

[project_PUT.json](Schemas_draft-03/Project/project_PUT.json)

[project_PATCH.json](Schemas_draft-03/Project/project_PATCH.json)

Modify a specific project, description similar to POST.


### 4.1.5 GET Project Extension Services ###

**Resource URL**

	GET /bcf/{version}/projects/{project_id}/extensions

[extensions_GET.json](Schemas_draft-03/Project/extensions_GET.json)

Retrieve a specific projects extensions.

**Example Request**

    https://example.com/bcf/1.0/projects/B724AAC3-5B2A-234A-D143-AE33CC18414/extensions


**Example Response**


    {
	"topic_type":
		[
			"Information",
			"Error"
		],
	"topic_status":
		[
			"Open",
			"Closed",
			"ReOpened"
		],
	"topic_label":
		[
			"Architecture",
			"Structural",
			"MEP"
		],
	"snippet_type":
		[
			".ifc",
			".csv"
		],
	"priority":
		[
			"Low",
			"Medium",
			"High"
		],
	"user_id_type":
		[
			"Architect@example.com",
			"BIM-Manager@example.com",
			"bob_heater@example.com"
		]
	}

### 4.1.6 POST Project Extension Services

**Resource URL**

	POST /bcf/{version}/projects/{project_id}/extensions

[extensions_POST.json](Schemas_draft-03/Project/extensions_POST.json)

Create a specific projects extensions.

Project extensions are used to define possible values that can be used in topics and comments, for example topic labels and priorities. They may change during the course of a project. The most recent extensions state which values are valid at a given moment for newly created topics and comments.

**Parameters**

JSON encoded body using the "application/json" content type.

<table border="1">
  <tr>
    <td>topic_type</td>
    <td>string array</td>
    <td>Enumeration of allowed values</td>
  </tr>
  <tr>
    <td>topic_status</td>
    <td>string array</td>
    <td>Enumeration of allowed values</td>
  </tr>
  <tr>
    <td>topic_label</td>
    <td>string array</td>
    <td>Enumeration of allowed values</td>
  </tr>
  <tr>
    <td>snippet_type</td>
    <td>string array</td>
    <td>Enumeration of allowed values</td>
  </tr>
  <tr>
    <td>priority</td>
    <td>string array</td>
    <td>Enumeration of allowed values</td>
  </tr>
  <tr>
    <td>user_id_type</td>
    <td>string array</td>
    <td>Enumeration of allowed values</td>
  </tr>
</table>

**Example Request**

    https://example.com/bcf/1.0/projects/B724AAC3-5B2A-234A-D143-AE33CC18414/extensions

	{
	"topic_type":
		[
			"Information",
			"Error"
		],
	"topic_status":
		[
			"Open",
			"Closed",
			"ReOpened"
		],
	"topic_label":
		[
			"Architecture",
			"Structural",
			"MEP"
		],
	"snippet_type":
		[
			".ifc",
			".csv"
		],
	"priority":
		[
			"Low",
			"Medium",
			"High"
		],
	"user_id_type":
		[
			"Architect@example.com",
			"BIM-Manager@example.com",
			"bob_heater@example.com"
		]
	}


**Example Response**


    {
	"topic_type":
		[
			"Information",
			"Error"
		],
	"topic_status":
		[
			"Open",
			"Closed",
			"ReOpened"
		],
	"topic_label":
		[
			"Architecture",
			"Structural",
			"MEP"
		],
	"snippet_type":
		[
			".ifc",
			".csv"
		],
	"priority":
		[
			"Low",
			"Medium",
			"High"
		],
	"user_id_type":
		[
			"Architect@example.com",
			"BIM-Manager@example.com",
			"bob_heater@example.com"
		]
	}


### 4.1.7 PUT Project Extension Services

**Resource URL**

	PUT /bcf/{version}/projects/{project_id}/extensions

[extensions_PUT.json](Schemas_draft-03/Project/extensions_PUT.json)

[extensions_PATCH.json](Schemas_draft-03/Project/extensions_PATCH.json)

Modify a specific projects extensions, description similar to POST.


---------

## 4.2 Topic Services ##

### 4.2.1 GET Topic Services ###

**Resource URL**
 
	GET /bcf/{version}/projects/{project_id}/topics

[topic_GET.json](Schemas_draft-03/Collaboration/Topic/topic_GET.json)

Retrieve a **collection** of topics related to a project (default sort order is creation_date).

**Odata filter parameters**
<table>
  <tr>
    <td>creation_author</td>
    <td>string</td>
    <td>UserID of the creation author (value from extension.xsd)</td>
  </tr>
  <tr>
    <td>modified_author</td>
    <td>string</td>
    <td>UserID of the modified author (value from extension.xsd)</td>
  </tr>
  <tr>
    <td>assigned_to</td>
    <td>string</td>
    <td>UserID of the assigned author (value from extension.xsd)</td>
  </tr>
  <tr>
    <td>topic_status</td>
    <td>string</td>
    <td>The status of a topic (value from extension.xsd)</td>
  </tr>
  <tr>
    <td>topic_type</td>
    <td>string</td>
    <td>The type of a topic (value from extension.xsd)</td>
  </tr>
  <tr>
    <td>creation_date</td>
    <td>datetime</td>
    <td>The creation date of a topic</td>
  </tr>
  <tr>
    <td>modified_date</td>
    <td>datetime</td>
    <td>The modification date of a topic</td>
  </tr>
  <tr>
    <td>labels</td>
    <td>collection of strings</td>
    <td>The labels of a topic (value from extension.xsd)</td>
  </tr>
</table>

**Odata sort parameters**
<table>
  <tr>
    <td>creation_date</td>
    <td>The creation date of a topic</td>
  </tr>
  <tr>
    <td>modified_date</td>
    <td>The modification date of a topic</td>
  </tr>
  <tr>
    <td>index</td>
    <td>The index of a topic</td>
  </tr>
</table>

**Example Request with odata**

Get topics that are open, assigned to Architect@example.com and created after December 5 2015. Sort the result on last modified

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics?$filter=assigned_to eq 'Architect@example.com' and status eq 'Open' and creation_date gt datetime'2015-12-05T00:00:00+01:00'&$orderby=modified_date desc

Odata does not support list operators. To achieve list query, use the 'or' operator. 
Get topics that have at least one of the labels 'Architecture', 'Structural' or 'Heating'

     https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics?$filter=contains(labels, 'Architecture') or contains(labels, 'Structural') or contains(labels, 'Heating')

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics

**Example Response**

    [
    {
        "guid": "A245F4F2-2C01-B43B-B612-5E456BEF8116",
        "title": "Example topic 1",
        "labels": [
            "Architecture",
            "Structural"
        ],
    	"creation_date": "2013-10-21T17:34:22.409Z"
    },
    {
        "guid": "A211FCC2-3A3B-EAA4-C321-DE22ABC8414",
        "title": "Example topic 2",
        "labels": [
            "Architecture",
            "Heating",
            "Electrical"
        ],
    	"creation_date": "2014-11-19T14:24:11.316Z"
    }
	]

### 4.2.2 POST Topic Services ###

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

### 4.2.3 GET Single Topic Services ###

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
            "Heating"],
		"assigned_to": "harry.muster@example.com",
        "bim_snippet":
				{
				"snippet_type": "clash",
				"is_external": true,
				"reference": "https://example.com/bcf/1.0/ADFE23AA11BCFF444122BB",
				"reference_schema": "https://example.com/bcf/1.0/clash.xsd"	
				}
    	}
 
### 4.2.4 PUT Single Topic Services ###

**Resource URL**

    PUT /bcf/{version}/projects/{project_id}/topics/{guid}

[topic_PUT.json](Schemas_draft-03/Collaboration/Topic/topic_PUT.json)

[topic_PATCH.json](Schemas_draft-03/Collaboration/Topic/topic_PATCH.json)

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



## 4.3 File Services ##


### 4.3.1 GET File (Header) Services ###

**Resource URL**

	GET /bcf/{version}/projects/{project_id}/topics/{guid}/files

[file_GET.json](Schemas_draft-03/Collaboration/File/file_GET.json)

Retrieve a **collection** of file references as topic header.

**Example Request**

	https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/files


**Example Response**

   	[
    	{
		"ifc_project": "0J$yPqHBD12v72y4qF6XcD",
		"file_name": "OfficeBuilding_Architecture_0001.ifc",
		"reference": "https://example.com/files/0J$yPqHBD12v72y4qF6XcD_0001.ifc"
    	},
	    {
		"ifc_project": "3hwBHP91jBRwPsmyf$3Hea",
		"file_name": "OfficeBuilding_Heating_0003.ifc",
		"reference": "https://example.com/files/3hwBHP91jBRwPsmyf$3Hea_0003.ifc"
    	}
	]

### 4.3.2 PUT File (Header) Services ###

**Resource URL**

	PUT /bcf/{version}/projects/{project_id}/topics/{guid}/files

[file_PUT.json](Schemas_draft-03/Collaboration/File/file_PUT.json)

[file_PATCH.json](Schemas_draft-03/Collaboration/File/file_PATCH.json)

Update a **collection** of file references on the topic header.

**Example Request**

	https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/files

    [
    	{
		"ifc_project": "0J$yPqHBD12v72y4qF6XcD",
		"file_name": "OfficeBuilding_Architecture_0001.ifc",
		"reference": "https://example.com/files/0J$yPqHBD12v72y4qF6XcD_0001.ifc"
    	},
	    {
		"ifc_project": "3hwBHP91jBRwPsmyf$3Hea",
		"file_name": "OfficeBuilding_Heating_0003.ifc",
		"reference": "https://example.com/files/3hwBHP91jBRwPsmyf$3Hea_0003.ifc"
    	}
	]

**Example Response**

    [
    	{
		"ifc_project": "0J$yPqHBD12v72y4qF6XcD",
		"file_name": "OfficeBuilding_Architecture_0001.ifc",
		"reference": "https://example.com/files/0J$yPqHBD12v72y4qF6XcD_0001.ifc"
    	},
	    {
		"ifc_project": "3hwBHP91jBRwPsmyf$3Hea",
		"file_name": "OfficeBuilding_Heating_0003.ifc",
		"reference": "https://example.com/files/3hwBHP91jBRwPsmyf$3Hea_0003.ifc"
    	}
	]

## 4.4 Comment Services ##

### 4.4.1 GET Comment Services ###

**Resource URL**

	GET /bcf/{version}/projects/{project_id}/topics/{guid}/comments

[comment_GET.json](Schemas_draft-03/Collaboration/Comment/comment_GET.json)

Retrieve a **collection** of all comments related to a topic (default ordering is date).

**Odata filter parameters**
<table>
  <tr>
    <td>author</td>
    <td>string</td>
    <td>UserID of the author (value from extension.xsd)</td>
  </tr>
  <tr>
    <td>status</td>
    <td>string</td>
    <td>The status of a comment (value from extension.xsd)</td>
  </tr>
  <tr>
    <td>date</td>
    <td>datetime</td>
    <td>The creation date of a comment</td>
  </tr>
</table>

**Odata sort parameters**
<table>
  <tr>
    <td>date</td>
    <td>The creation date of a comment</td>
  </tr>
</table>

**Example Request with odata**

Get comments that are closed and created after December 5 2015. Sort the result on first created

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/comments?$filter=status eq 'Closed' and date gt datetime'2015-12-05T00:00:00+01:00'&$orderby=date asc

**Example Request**

    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics

**Example Request**

	https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/comments

**Example Response**

    [
    {
        "guid": "C4215F4D-AC45-A43A-D615-AA456BEF832B",
		"status": "open",
    	"date": "2013-10-21T17:34:22.409Z",
		"author": "max.muster@example.com",
		"comment": "Clash found",
		"topic_guid": "B345F4F2-3A04-B43B-A713-5E456BEF8228"
		
    },
    {
        "guid": "A333FCA8-1A31-CAAC-A321-BB33ABC8414",
		"status": "closed",
    	"date": "2013-11-19T14:24:11.316Z",
		"author": "bob.heater@example.com",		
		"comment": "will rework the heating model",
		"topic_guid": "B345F4F2-3A04-B43B-A713-5E456BEF8228"
    }
	]

### 4.4.2 POST Comment Services ###

**Resource URL**

	POST /bcf/{version}/projects/{project_id}/topics/{guid}/comments

[comment_POST.json](Schemas_draft-03/Collaboration/Comment/comment_POST.json)

Add a new comment to a topic.

**Parameters**

JSON encoded body using the "application/json" content type.


<table border="1">
  <tr>
    <td>verbal_status</td>
    <td>string</td>
    <td>The verbal status of a comment (possible values from extension.xsd)</td>
    <td>optional</td>
  </tr>
  <tr>
    <td>status</td>
    <td>string</td>
    <td>The status of a comment (default unknown)</td>
    <td>mandatory</td>
  </tr>
  <tr>
    <td>comment</td>
    <td>string</td>
    <td>The comment</td>
    <td>mandatory</td>
  </tr>
  <tr>
    <td>viewpoint_guid</td>
    <td>string</td>
    <td>The GUID of the related viewpoint</td>
    <td>optional</td>
  </tr>
  <tr>
    <td>reply_to_comment_guid</td>
    <td>string</td>
    <td>GUID of the comment to which this comment replies to</td>
    <td>optional</td>
  </tr>
</table>

**Example Request**

	https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/comments

    {
		"verbal_status": "closed",
		"status": "closed",	
		"comment": "will rework the heating model",
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

### 4.4.3 GET Single Comment Services ###

**Resource URL**

	GET /bcf/{version}/projects/{project_id}/topics/{guid}/comments/{guid}

[comment_GET.json](Schemas_draft-03/Collaboration/Comment/comment_GET.json)

Get a single comment.

**Example Request**

	https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/comments/A333FCA8-1A31-CAAC-A321-BB33ABC8414

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

### 4.4.4 PUT Single Comment Services ###

**Resource URL**

	PUT /bcf/{version}/projects/{project_id}/topics/{guid}/comments/{guid}

[comment_PUT.json](Schemas_draft-03/Collaboration/Comment/comment_PUT.json)

[comment_PATCH.json](Schemas_draft-03/Collaboration/Comment/comment_PATCH.json)

Update a single comment, description similar to POST.

## 4.5 Viewpoint Services ##

### 4.5.1 GET Viewpoint Services ###

**Resource URL**

	GET /bcf/{version}/projects/{project_id}/topics/{guid}/viewpoints

[viewpoint_GET.json](Schemas_draft-03/Collaboration/Viewpoint/viewpoint_GET.json)

Retrieve a **collection** of all viewpoints related to a topic.

**Example Request**

	https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/viewpoints

**Example Response**

	[
		{  
   		"guid":"b24a82e9-f67b-43b8-bda0-4946abf39624",
   		"perspective_camera":{
     		"camera_view_point":{  
        		"x":0.0,
        		"y":0.0,
         		"z":0.0},  
     		"camera_direction":{  
        		"x":1.0,
        		"y":1.0,
         		"z":2.0},
      		"camera_up_vector":{  
	    		"x":0.0,
	    		"y":0.0,
          		"z":1.0},
      		"field_of_view":90.0},
   		"lines":{  
      		"line":[  
         			{  
            		"start_point":{  
              	 		"x":2.0,
               			"y":1.0,
               			"z":1.0},
           			"end_point":{  
               			"x":0.0,
			   			"y":1.0,
			   			"z":0.7}
         			}
      			   ]},
   		"clipping_planes":{  
      		"clipping_plane":[  
         			{  
            		"location":{  
               			"x":0.7,
               			"y":0.3,
               			"z":-0.2},
            		"direction":{  
               			"x":1.0,
			   			"y":0.4,
			   			"z":0.1}
         			}
      				]
   				}
			},
		{  
   		"guid":"a11a82e7-e66c-34b4-ada1-5846abf39133",
   		"perspective_camera":{
     		"camera_view_point":{  
        		"x":0.0,
        		"y":0.0,
         		"z":0.0},  
     		"camera_direction":{  
        		"x":1.0,
        		"y":1.0,
         		"z":2.0},
      		"camera_up_vector":{  
	    		"x":0.0,
	    		"y":0.0,
          		"z":1.0},
      		"field_of_view":90.0},
   		"lines":{  
      		"line":[  
         			{  
            		"start_point":{  
              	 		"x":1.0,
               			"y":1.0,
               			"z":1.0},
           			"end_point":{  
               			"x":0.0,
			   			"y":0.0,
			   			"z":0.0}
         			}
      			   ]},
   		"clipping_planes":{  
      		"clipping_plane":[  
         			{  
            		"location":{  
               			"x":0.5,
               			"y":0.5,
               			"z":0.5},
            		"direction":{  
               			"x":1.0,
			   			"y":0.0,
			   			"z":0.0}
         			}
      				]
   				}
			}
	]

### 4.5.2 POST Viewpoint Services ###

**Resource URL**

	POST /bcf/{version}/projects/{project_id}/topics/{guid}/viewpoints

[viewpoint_POST.json](Schemas_draft-03/Collaboration/Viewpoint/viewpoint_POST.json)

Add a new viewpoint.

**Parameters**

JSON encoded body using the "application/json" content type.


<table border="1">
  <tr>
    <td>x, y, z</td>
    <td>number</td>
    <td>Numbers defining ether a point or a vector </td>
    <td>optional</td>
  </tr>
  <tr>
    <td>orthogonal camera</td>
    <td>element</td>
    <td>Orthogonal camera view</td>
    <td>optional</td>
  </tr>
  <tr>
    <td>camera_view_point</td>
    <td>element</td>
    <td>Viewpoint of the camera</td>
    <td>optional</td>
  </tr>
  <tr>
    <td>camera_directiont</td>
    <td>element</td>
    <td>Direction of the camera</td>
    <td>optional</td>
  </tr>
  <tr>
    <td>camera_up_vector</td>
    <td>element</td>
    <td>Direction of camera up</td>
    <td>optional</td>
  </tr>
  <tr>
    <td>view_to_world_scale</td>
    <td>element</td>
    <td>Proportion of camera view to model</td>
    <td>optional</td>
  </tr>
  <tr>
    <td>perspective camera</td>
    <td>element</td>
    <td>Perspective view of the camera</td>
    <td>optional</td>
  </tr>
  <tr>
    <td>camera_view_point</td>
    <td>element</td>
    <td>Viewpoint of the camera</td>
    <td>optional</td>
  </tr>
  <tr>
    <td>camera_directiont</td>
    <td>element</td>
    <td>Direction of the camera</td>
    <td>optional</td>
  </tr>
  <tr>
    <td>camera_up_vector</td>
    <td>element</td>
    <td>Direction of camera up</td>
    <td>optional</td>
  </tr>
  <tr>
    <td>field_of_view</td>
    <td>element</td>
    <td>Field of view</td>
    <td>optional</td>
  </tr>
 <tr>
    <td>line</td>
    <td>element</td>
    <td>A graphical line</td>
    <td>optional</td>
  </tr>
  <tr>
    <td>start_point</td>
    <td>element</td>
    <td>Start point of the line</td>
    <td>optional</td>
  </tr>
  <tr>
    <td>end_point</td>
    <td>element</td>
    <td>end point of the line</td>
    <td>optional</td>
  </tr>
 <tr>
    <td>clipping_plane</td>
    <td>element</td>
    <td>Clipping plane for the model view</td>
    <td>optional</td>
  </tr>
  <tr>
    <td>location</td>
    <td>element</td>
    <td>Origin of the clipping plane</td>
    <td>optional</td>
  </tr>
  <tr>
    <td>direction</td>
    <td>element</td>
    <td>direction of the clipping plane</td>
    <td>optional</td>
  </tr>
  <tr>
    <td>bitmaps</td>
    <td>array</td>
    <td>Array of embedded pictures in the viewpoint</td>
    <td>optional</td>
  </tr>
  <tr>
    <td>guid</td>
    <td>string</td>
    <td>Guid for the bitmap</td>
    <td>mandatory</td>
  </tr>
  <tr>
    <td>bitmap_type</td>
    <td>enum</td>
    <td>Format of the bitmap. Predefined values "png" = 0, "jpg" = 1, "bmp" = 2</td>
    <td>mandatory</td>
  </tr>

  <tr>
    <td>location</td>
    <td>element</td>
    <td>Location of the center of the bitmap in world coordinates (point)</td>
    <td>optional</td>
  </tr>
  <tr>
    <td>normal</td>
    <td>element</td>
    <td>Normal vector of the bitmap (vector)</td>
    <td>optional</td>
  </tr>
  <tr>
    <td>up</td>
    <td>element</td>
    <td>Up vector of the bitmap (vector)</td>
    <td>optional</td>
  </tr>
</table>

**Example Request**

	https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/viewpoints
	{  
   		"perspective_camera":{
     		"camera_view_point":{  
        		"x":0.0,
        		"y":0.0,
         		"z":0.0},  
     		"camera_direction":{  
        		"x":1.0,
        		"y":1.0,
         		"z":2.0},
      		"camera_up_vector":{  
	    		"x":0.0,
	    		"y":0.0,
          		"z":1.0},
      		"field_of_view":90.0},
   		"lines":{  
      		"line":[  
         			{  
            		"start_point":{  
              	 		"x":1.0,
               			"y":1.0,
               			"z":1.0},
           			"end_point":{  
               			"x":0.0,
			   			"y":0.0,
			   			"z":0.0}
         			}
      			   ]},
   		"clipping_planes":{  
      		"clipping_plane":[  
         			{  
            		"location":{  
               			"x":0.5,
               			"y":0.5,
               			"z":0.5},
            		"direction":{  
               			"x":1.0,
			   			"y":0.0,
			   			"z":0.0}
         			}
      				]
   				}
			}

**Example Response**

	{  
   		"guid":"a11a82e7-e66c-34b4-ada1-5846abf39133",
   		"perspective_camera":{
     		"camera_view_point":{  
        		"x":0.0,
        		"y":0.0,
         		"z":0.0},  
     		"camera_direction":{  
        		"x":1.0,
        		"y":1.0,
         		"z":2.0},
      		"camera_up_vector":{  
	    		"x":0.0,
	    		"y":0.0,
          		"z":1.0},
      		"field_of_view":90.0},
   		"lines":{  
      		"line":[  
         			{  
            		"start_point":{  
              	 		"x":1.0,
               			"y":1.0,
               			"z":1.0},
           			"end_point":{  
               			"x":0.0,
			   			"y":0.0,
			   			"z":0.0}
         			}
      			   ]},
   		"clipping_planes":{  
      		"clipping_plane":[  
         			{  
            		"location":{  
               			"x":0.5,
               			"y":0.5,
               			"z":0.5},
            		"direction":{  
               			"x":1.0,
			   			"y":0.0,
			   			"z":0.0}
         			}
      				]
   				}
			}

### 4.5.3 GET Single Viewpoint Services ###

**Resource URL**

    GET /bcf/{version}/projects/{guid}/topics/{guid}/viewpoints/{guid}

[viewpoint_GET.json](Schemas_draft-03/Collaboration/Viewpoint/viewpoint_GET.json)


Retrieve a specific viewpoint.

**Example Request**


    https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/viewpoints/a11a82e7-e66c-34b4-ada1-5846abf39133


**Example Response**

	{  
   		"guid":"a11a82e7-e66c-34b4-ada1-5846abf39133",
   		"perspective_camera":{
     		"camera_view_point":{  
        		"x":0.0,
        		"y":0.0,
         		"z":0.0},  
     		"camera_direction":{  
        		"x":1.0,
        		"y":1.0,
         		"z":2.0},
      		"camera_up_vector":{  
	    		"x":0.0,
	    		"y":0.0,
          		"z":1.0},
      		"field_of_view":90.0},
   		"lines":{  
      		"line":[  
         			{  
            		"start_point":{  
              	 		"x":1.0,
               			"y":1.0,
               			"z":1.0},
           			"end_point":{  
               			"x":0.0,
			   			"y":0.0,
			   			"z":0.0}
         			}
      			   ]},
   		"clipping_planes":{  
      		"clipping_plane":[  
         			{  
            		"location":{  
               			"x":0.5,
               			"y":0.5,
               			"z":0.5},
            		"direction":{  
               			"x":1.0,
			   			"y":0.0,
			   			"z":0.0}
         			}
      				]
   				}
			}


### 4.5.4 PUT Single Viewpoint Services

**Resource URL**

    PUT /bcf/{version}/projects/{guid}/topics/{guid}/viewpoints/{guid}

[viewpoint_PUT.json](Schemas_draft-03/Collaboration/Viewpoint/viewpoint_PUT.json)

[viewpoint_PATCH.json](Schemas_draft-03/Collaboration/Viewpoint/viewpoint_PATCH.json)

Update a single viewpoint, description similar to POST.


### 4.5.5 GET snapshot of a Viewpoint Service ###

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

## 4.6 Component Services ##

### 4.6.1 GET Component Services ###

**Resource URL**

	GET /bcf/{version}/projects/{project_id}/topics/{guid}/viewpoints/{guid}/components

[component_GET.json](Schemas_draft-03/Collaboration/Component/component_GET.json)

Retrieve a **collection** of all components related to a viewpoint.

**Example Request**

	https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/viewpoints/a11a82e7-e66c-34b4-ada1-5846abf39133/components

**Example Response**

	[
 	{
  		"ifc_guid":"2MF28NhmDBiRVyFakgdbCT",
  		"selected": true,
  		"visible":  true,
  		"color":"0A00FF",
  		"originating_system":"Example CAD Application",
  		"authoring_tool_id":"EXCAD/v1.0"
 	},
 	{
  		"ifc_guid":"3$cshxZO9AJBebsni$z9Yk",
  		"selected": true,
  		"visible":  true,
  		"color":"0A00FF",
  		"originating_system":"Example CAD Application",
  		"authoring_tool_id":"EXCAD/v1.0"
 	}
	]

### 4.6.2 PUT Component Services ###

**Resource URL**

	POST /bcf/{version}/projects/{project_id}/topics/{guid}/viewpoints/{guid}/components

[component_PUT.json](Schemas_draft-03/Collaboration/Component/component_PUT.json)

[component_PATCH.json](Schemas_draft-03/Collaboration/Component/component_PATCH.json)

Add or update a **collection** of all components related to a viewpoint.

**Example Request**

	https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/viewpoints/a11a82e7-e66c-34b4-ada1-5846abf39133/components

	[
 	{
  		"ifc_guid":"2MF28NhmDBiRVyFakgdbCT",
  		"selected": true,
  		"visible":  true,
  		"color":"0A00FF",
  		"originating_system":"Example CAD Application",
  		"authoring_tool_id":"EXCAD/v1.0"
 	},
 	{
  		"ifc_guid":"3$cshxZO9AJBebsni$z9Yk",
  		"selected": true,
  		"visible":  true,
  		"color":"0A00FF",
  		"originating_system":"Example CAD Application",
  		"authoring_tool_id":"EXCAD/v1.0"
 	}
	]

**Example Response**
	
	[
 	{
  		"ifc_guid":"2MF28NhmDBiRVyFakgdbCT",
  		"selected": true,
  		"visible":  true,
  		"color":"0A00FF",
  		"originating_system":"Example CAD Application",
  		"authoring_tool_id":"EXCAD/v1.0"
 	},
 	{
  		"ifc_guid":"3$cshxZO9AJBebsni$z9Yk",
  		"selected": true,
  		"visible":  true,
  		"color":"0A00FF",
  		"originating_system":"Example CAD Application",
  		"authoring_tool_id":"EXCAD/v1.0"
 	}
	]


## 4.7 Related Topics Services ##

### 4.7.1 GET Related Topics Services ###

**Resource URL**

	GET /bcf/{version}/projects/{project_id}/topics/{guid}/related_topics

[related_topic_GET.json](Schemas_draft-03/Collaboration/RelatedTopic/related_topic_GET.json)

Retrieve a **collection** of all related topics to a topic.

**Example Request**

	https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/related_topics

**Example Response**

	[
 	{
  		"related_topic_guid":"db49df2b-0e42-473b-a3ee-f7b785d783c4",
 	},
 	{
  		"related_topic_guid":"6963a846-54d1-4050-954d-607cd5e48aa3",
 	}
	]

### 4.7.2 PUT Related Topics Services ###

**Resource URL**

	POST /bcf/{version}/projects/{project_id}/topics/{guid}/related_topics

[related_topic_PUT.json](Schemas_draft-03/Collaboration/RelatedTopic/related_topic_PUT.json)

[related_topic_PATCH.json](Schemas_draft-03/Collaboration/RelatedTopic/related_topic_PATCH.json)

Add or update a **collection** of all related topics to a topic.

**Example Request**

	https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/related_topics

	[
 	{
  		"related_topic_guid":"db49df2b-0e42-473b-a3ee-f7b785d783c4"
 	},
 	{
  		"related_topic_guid":"6963a846-54d1-4050-954d-607cd5e48aa3"
 	},
	{
  		"related_topic_guid":"bac66ab4-331e-4f21-a28e-083d2cf2e796"
 	}
	]

**Example Response**

	[
 	{
  		"related_topic_guid":"db49df2b-0e42-473b-a3ee-f7b785d783c4"
 	},
 	{
  		"related_topic_guid":"6963a846-54d1-4050-954d-607cd5e48aa3"
 	},
	{
  		"related_topic_guid":"bac66ab4-331e-4f21-a28e-083d2cf2e796"
 	}
	]

## 4.8 Document Reference Services ##

### 4.8.1 GET Document Reference Services ###

**Resource URL**

	GET /bcf/{version}/projects/{project_id}/topics/{guid}/document_references

[document_reference_GET.json](Schemas_draft-03/Collaboration/DocumentReference/document_reference_GET.json)

Retrieve a **collection** of all document references to a topic.

**Example Request**

	https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/document_references

**Example Response**

	[
 	{
  		"guid":"472ab37a-6122-448e-86fc-86503183b520",
  		"referenced_document":"http://example.com/files/LegalRequirements.pdf",
  		"description":"The legal requirements for buildings."
 	},
 	{
	  	"guid":"6cbfe31d-95c3-4f4d-92a6-420c23698721",
	  	"referenced_document":"http://example.com/files/DesignParameters.pdf",
	  	"description":"The building owners global design parameters for buildings."
 	}
	]

### 4.8.2 PUT Document Reference Services ###

**Resource URL**

	POST /bcf/{version}/projects/{project_id}/topics/{guid}/document_references

[document_reference_PUT.json](Schemas_draft-03/Collaboration/DocumentReference/document_reference_PUT.json)

[document_reference_PATCH.json](Schemas_draft-03/Collaboration/DocumentReference/document_reference_PATCH.json)

Add or update document references to a topic.
The PUT object may either contain the property "guid" to reference a document stored on the BCF server (see section 4.9) OR the property "referenced_document" to point to an external resource.

**Example Request**

	https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/topics/B345F4F2-3A04-B43B-A713-5E456BEF8228/document_references

	[
 	{
  		"referenced_document":"http://example.com/files/LegalRequirements.pdf",
  		"description":"The legal requirements for buildings."
 	}
	]

**Example Response**

	[
 	{
  		"guid":"472ab37a-6122-448e-86fc-86503183b520",
  		"referenced_document":"http://example.com/files/LegalRequirements.pdf",
  		"description":"The legal requirements for buildings."
 	}
	]

## 4.9 Document Services

### 4.9.1 GET Documents Services

[document_GET.json](Schemas_draft-03/Collaboration/Document/document_GET.json)

**Resource URL**

	GET /bcf/{version}/projects/{project_id}/documents

Retrieve a **collection** of all documents uploaded to a project.

**Example Request**

	https://example.com/bcf/1.0/projects/F445F4F2-4D02-4B2A-B612-5E456BEF9137/documents

**Example Response**

	[
 	{
  		"guid":"472ab37a-6122-448e-86fc-86503183b520",
  		"filename":"LegalRequirements.pdf"
 	},
 	{
	  	"guid":"6cbfe31d-95c3-4f4d-92a6-420c23698721",
	  	"filename":"DesignParameters.pdf"
 	}
	]

### 4.9.2 POST Document Services


**Resource URL**

	POST /bcf/{version}/projects/{project_id}/documents

Upload a document (binary file) to a project. The following HTTP headers are used for the upload:

	Content-Type: application/octet-stream;
	Content-Length: {Size of file in bytes};
	Content-Disposition: attachment; filename="{Filename.extension}";

**Example Response**

 	{
  		"guid":"472ab37a-6122-448e-86fc-86503183b520",
		"filename":"Official_Building_Permission.pdf"
 	}

### 4.9.3 GET Document Services

**Resource URL**

	GET /bcf/{version}/projects/{project_id}/documents/{guid}

Retrieves a document as binary file. Will use the following HTTP headers to deliver additional information:

	Content-Type: application/octet-stream;
	Content-Length: {Size of file in bytes};
	Content-Disposition: attachment; filename="{Filename.extension}";


