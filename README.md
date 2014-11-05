## BCF	 REST API 
![](https://raw.githubusercontent.com/BuildingSMART/BCF/master/Icons/BCFicon128.png)

**Version 1.0** based on BCFv2.
[GitHub repository](https://github.com/BuildingSMART/BCF-API)



----------



# 1. Introduction #

BCF is a format for managing issues on a BIM project. RESTful BCF-API supports the exchange of BCFv2 issues between software applications.

All API access is over HTTPS. Data is sent as query parameters and received as JSON. Every resource has a corresponding Json Schema. (There are also XSD-Schemas available but XML support ist optionally). Json Hyper Schema is used for link definition. Authentication method is OAuth2.



## 1.1 Paging, Sorting, Filtering ##


- Link header introduced by RFC 5988
- custom HTTP header  X-Total-Count (HEAD)

***page, page_size, sort***

Only „descending“ if property exits.
 
If no descending property is there ->  ascending=true

if descending=true -> descending

if descending=false -> ascending

***"&"** to combine different query parameters (operations)*



Example 1:


    .../bcf/{version}/projects/{guid}/topics?page=1&page_size=5&sort=priority&descending=true

Example 2:

    .../bcf/{version}/projects/{guid}/topics?page=1&page_size=5&sort=priority&descending=true&filter=(label%3DArchitecture%7Clabel%3DStructural)%26topicstatus%21%3DClosed




[Filter use escape characters](http://www.december.com/html/spec/esccodes.html)
  



## 1.2 Caching ##

ETags, or entity-tags, are an important part of HTTP, being a critical part of caching, and also used in "conditional" requests.
The ETag response-header field value, an entity tag, provides for an "opaque" cache validator.
The easiest way to think of an etag is as an MD5 or SHA1 hash of all the bytes in a representation. If just one byte in the representation changes, the etag will change.

ETags are returned in a response to a GET:


    
    joe@joe-laptop:~$ curl --include http://bitworking.org/news/
    
    HTTP/1.1 200 Ok
    
    Date: Wed, 21 Mar 2007 15:06:15 GMT
    
    Server: Apache
    
    etag: "078de59b16c27119c670e63fa53e5b51"
    
    Content-Length: 23081
    …..
  

## 1.3 Cross origin resource sharing (Cors) ##

The server will put the "Access-Control-Allow-Headers" in the response header and specify who can access the server(json) resources. The client can look for this value and proceed with accessing the resources. 

The server has  a web config file .. "*" means the server allow the resources for all domains.

    <httpProtocol>
      <customHeaders>
    	<add name="Access-Control-Allow-Headers" value="Content-Type, Accept, X-Requested-With,  Authorization" />
    	<add name="Access-Control-Allow-Methods" value="GET, POST, PUT, DELETE, OPTIONS" />
    	<add name="Access-Control-Allow-Origin" value="*" />
      </customHeaders>
     </httpProtocol>


## 1.4 Http status codes ##

-   200 OK (Data is returned)
-   201 No content (Data has been deleted)
-   302 Redirect (Returning a redirect to the GET-resource for the data that has been created/updated)
-   400 BadRequest (Input data is invalid)
-   401 Unauthorized (User don’t have access to the requested resource)
-   403 Forbidden
-   404 Not found (It must be discussed if the user should get “unauthorized” to resources he don’t have access to, or “not found")
-   422 Unprocessable entity (Input data is well formed, but the semantic is wrong; Example: Resource define that a value cannot be “null”, but the value is “null”)

## 1.5 Error response body format ##

BCF-API has a specified error response body format [error.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/error.json).

----------




# Topology 1 - BCF-Server only#

Model collaboration is managed through a shared file server or a network file sharing service like Dropbox. The BCF-Server handels the Authentication and the BCF-Issues. 

![Topology1](Images/Topology1.png)



# Topology 2 - Colocated BCF-Server and Model Server#

BCF and model server are co located on the same hosts.


![Topology3](Images/Topology3.png)


----------


## Information Services ##

[version.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/version.json)

**Recource URL (public resource)**

`GET /bcf/version` 


**Parameters**


<table border="1">
  <tr>
    <td>version_id</td>
    <td>string</td>
    <td>ID of the version</td>
  </tr>
  <tr>
    <td>detailed_version</td>
    <td>string</td>
    <td>URL to version on Github</td>
  </tr>
  <tr>
    <td>link_schemas</td>
    <td>string</td>
    <td>URL to schemas on Github</td>
  </tr>
</table>


---------- 


## Authentication ##

[auth.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/auth.json)

Authentication is based on the [OAuth 2.0 Protocol](http://tools.ietf.org/html/draft-ietf-oauth-v2-22).

**Recource URL (public resource)**

    GET /bcf/auth	

**Parameters**

<table border="1">
  <tr>
    <td>oauth2_auth_url</td>
    <td>string</td>
    <td>URL to authorisation page</td>
  </tr>
  <tr>
    <td>oauth2_token_url</td>
    <td>string</td>
    <td>URL for token requests</td>
  </tr>
</table>


**Example Request**

    https://.../bcf/auth	

**Example Response**

	{
	"oauth2_auth_url": "https://bim--it.net/bcf/oauth2/auth", 
	"oauth2_token_url": "https://bim--it.net/bcf/oauth2/token" 
	}


**Oauth2 protocol flow - Client Request -**

The Client uses the **"oauth2\_auth_url"** and adds the following parameters to it.


1. response_type -> code
2. client_id -> your client\_id
3. state -> unique user defined value

Example Url:

	https://bim--it.net/bcf/oauth2/auth?response_type=code&client_id=<YourClientID>&state=D98F9B4F-5B0E-4948-B8B5-59F4FE23B8E0


Tip:
You can use the state parameter to transport custom information. 

Open a browser window or redirect the user to this resource. This redirects back to the specified redirect URI with the provided state and the authorization code as a query parameter if the user allows your app to access the account, the value "access_denied" in the error query parameter if the user denies access.

**Oauth2 protocol flow - Token Request -**

[token.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/token.json)

The Client uses the **"oauth2\_token_url"** to request a token.

	POST https://bim--it.net/bcf/oauth2/token

Content type: application/x-www-form-urlencoded. 

The POST request has to be done via HTTP Basic Authorization. Your "ClientID" is the username, your "ClientSecret" is the password.

Post Request Body: 

	grant_type=authorization_code&code=<YourAccessCode>


The access token will be returned as JSON in the response body and is an arbitrary string, guarantied to fit in a varchar(255) field.




**Oauth2 protocol flow - Refresh Token Request -**

[token.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/token.json)


The process to retrieve a refresh token is exactly the same as retrieving a token except the Post Request Body.

Post Request Body:

	grant_type=refresh_token&refresh_token=<YourRefreshToken>

The refresh token can only be used once to retrieve a token and a new refresh token. 


**Oauth2 protocol flow - Requesting Resources -**

When requesting other resources the access token must be passed via the Authorization header using the Bearer scheme *(e.g. Authorization: Bearer T9UNRV4sC9vr7ga)*.

----------



## Project Services ##


[project.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/project.json), [extensions.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/extensions.json)

**Recource URL**

    GET /bcf/{version}/projects

[GET_project.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/GET_project.json)

Retrieve a list of projects where the currently logged on user is assigned to.


**Example Request**

    https://.../bcf/1.0/projects

**Example Response**


    [{
        "project_id": "9a1e6b9a668740b88b076a08fcca2b53",
		"name": "Example project 1"
    },
    {
        "project_id": "4950fb76d0bd4b98b3a7c2152bd62b79"
		"name": "Example project 2"
    }]


----------
**Recource URL**

    POST /bcf/{version}/projects

Add a new project

**Parameters**

JSON encoded body using the "application/x-www-form-urlencoded" content type.

<table border="1">

  <tr>
    <td>name</td>
    <td>string</td>
    <td>The project name</td>
  </tr>
</table>


**Example Request**

    https://.../bcf/1.0/projects
	{
    "name": "Example project 3"
	}

**Example Response**


    {
      "name": "Example project 3",
      "id": "dabc6dfce6e849bfada976d9fa9e294a"
    }



----------

**Recource URL**

    GET /bcf/{version}/projects/{project_id}

Retrieve a specific project

**Example Request**

    https://.../bcf/1.0/projects/dabc6dfce6e849bfada976d9fa9e294a


----------


    GET, PUT, DELETE /bcf/{version}/projects/{project_id}

- GET - Retrieve a specific project
- PUT - Modify a specific project
- DELETE - Delete a specific project




    GET, POST, PUT, DELETE /bcf/{version}/projects/{project_id}/extension

- GET - Retrieve the project extension schema
- POST - Add the project extension schema
- PUT - Change the project extension schema
- DELETE – Delete the project extension schema

Long URL:  /bcf/{version}/teams/{guid}/projects/{project_id}/extensions


---------


## BCF Services ##

#### *Topic* ####
[topic.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/topic.json)


    GET, POST /bcf/{version}/projects/{project_id}/topics

- GET - Retrieve topics of a project (default sort = CreationDate)
- POST - Add a new topic to a project

Long URL: -

    GET, PUT, DELETE /bcf/{version}/topics/{guid}

- GET - Retrieve a specific topic
- PUT - Update a specific topic
- DELETE - Delete a specific topic

Long URL: /bcf/{version}/projects/{project_id}/topics/{guid}


----------



#### *File* ####
 [file.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/file.json)

`GET /bcf/{version}/topics/{guid}/files`

- GET - Retrieve the header of a topic
- POST - Assign a file to a topic

Long URL: /{version}/projects/{project_id}/topics/{guid}/files

`DELETE /bcf/{version}/topic/{guid}/files/{reference}`

- DELETE - Remove a file from topic header

Long URL: /bcf/{version}/projects/{project_id}/topics/{guid}/revisions/{reference}


----------


#### *Comment* ####
 [comment.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/comment.json)


    GET, POST /bcf/{version}/topics/{guid}/comments

- GET - Retrieve comments of a topic
- POST - Add a new comment to a topic

Long URL: /bcf/{version}/projects/{project_id}/topics/{guid}/comments

    GET, PUT, DELETE /bcf/{version}/comments/{guid}

- GET - Retrieve a specific comment
- PUT - Update a specific comment
- DELETE - Delete a specific comment

Long URL: /bcf/{version}/projects/{project_id}/topics/{guid}/comments/{guid}

    GET, POST, DELETE /bcf/{version}/comments/{guid}/viewpoint

- GET - Retrieve the viewpoint assigned to a comment
- POST - Assign a viewpoint to a comment
- DELETE - Delete the viewpoint assigned to a comment

Long URL: /bcf/{version}/projects/{project_id}/topics/{guid}/comments/{guid}/viewpoint

    GET, POST, DELETE /bcf/{version}/comments/{guid}/reply_to

- GET - Retrieve the replyTo comment related to a comment
- POST - Add a replyTo comment relation to a comment
- DELETE - Delete the replyTo comment relation on a comment

Long URL: /bcf/{version}/projects/{project_id}/topics/{guid}/comments/{guid}/reply_to


----------


#### *Viewpoint* ####
 [viewpoint.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/viewpoint.json)


    GET, POST /{version}/topics/{guid}/viewpoints


- GET - Retrieve viewpoints of a topic
- POST - Add a new viewpoint to a topic

Long URL: /{version}/projects/{project_id}/topics/{guid}/viewpoints


    GET, PUT, DELETE /{version}/viewpoints/{guid}

- GET - Retrieve a specific viewpoint
- PUT - Modify a specific viewpoint
- DELETE – Delete a specific viewpoint

Long URL: /{version}/projects/{project_id}/topics/{guid}/viewpoints/{guid}


    GET, POST, DELETE /{version}/viewpoints/{guid}/bitmap

- GET - Retrieve the bitmap related to a viewpoint
- PUT - Add a bitmap to the viewpoint
- DELETE – Delete the bitmap of the viewpoint

Long URL: /{version}/projects/{project_id}/topics/{guid}/viewpoints/{guid}/bitmap

#### *Component* ####
 [component.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/component.json)


    GET, POST /{version}/viewpoints/{guid}/components

- GET - Retrieve components of a viewpoint
- POST - Add a new component to a viewpoint

Long URL: /{version}/projects/{project_id}/topics/{guid}/viewpoints/{guid}/components

    GET, PUT, DELETE /{version}/components/{ifc_guid}

- GET - Retrieve a specific component
- PUT - Modify a specific component
- DELETE – Delete a specific component

Long URL: /{version}/projects/{project_id}/topics/{guid}/viewpoints/{guid}/components/{ifc_guid}


----------


#### *Related Topic* ####
 [related_topic.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/related_topic.json)

    GET, POST /{version}/topics/{guid}/related_topics

- GET - Retrieve related topics to a topic
- POST - Add a new related_topic to a topic

Long URL: /{version}/projects/{project_id}/topics/{guid}/related_topics

    DELETE /{version}/topics/{guid}/related_topics/{guid}

- DELETE – Delete related topic

Long URL: /{version}/projects/{project_id}/topics/{guid}/related_topics/{guid}


----------



#### *Document Reference* ####
 [document_reference.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/document_reference.json)


    GET, POST /{version}/topics/{guid}/document_references

- GET - Retrieve documents referenced on a topic
- POST - Add a new document reference to a topic

Long URL: /{version}/projects/{project_id}/topics/{guid}/document_references

    DELETE /{version}/topics/{guid}/document_references/{guid}

- GET - Retrieve a document referenced on a topic
- DELETE – Delete a document reference

Long URL: /{version}/projects/{project_id}/topics/{guid}/document_references/{guid} 


----------


