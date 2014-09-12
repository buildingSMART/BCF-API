## BIM REST API 
![](https://raw.githubusercontent.com/BuildingSMART/BCF/master/Icons/BCFicon128.png)

**Version 0.99** based on BCFv2.
[GitHub repository](https://github.com/BuildingSMART/BCF-API)


Services:

- Information
- Team, Project, Domain
- Revision
- BCF
- Attachment
- User


----------



# 1. Introduction #


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


    .../v0.99/projects/{guid}/topics?page=1&page_size=5&sort=priority&descending=true

Example 2:

    .../v0.99/projects/{guid}/topics?page=1&page_size=5&sort=priority&descending=true&filter=(label%3DArchitecture%7Clabel%3DStructural)%26topicstatus%21%3DClosed




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




# 2. Services #



## 2.1 Information Services ##

[version.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/version.json), [link.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/link.json), [colors.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/colors.json)

Services:

`GET /version` 

- GET - Retrieve BCF-Version

---------- 


    GET /V0.99/schemas

- GET - Retrieve link schema that contains all available schemas in BIM-API

---------- 


    GET /V0.99/schemas/{rel}

- GET - Retrieve a specific schema.

----------


`GET /V0.99/colors` 

- GET - Retrieve ARGB values for colours


----------

## 2.2 Authentication ##

Authentication is based on the [OAuth 2.0 Protocol](http://tools.ietf.org/html/draft-ietf-oauth-v2-22).

Services:

    GET /V0.99/oauth/authorize

Open a browser window or redirect the user to this resource.

Redirects back to the specified redirect URI with the provided state as a query parameter or either...





- an authorization code as a query parameter if the user allows your app to access the account.
- the value "access_denied" in the error query parameter if the user denies access.

Long Url: -


    POST /V0.99/oauth/access_token

After you have received the authorization code you can request an access token. The access token will be returned as JSON in the response body.

The access token is an arbitrary string, guarantied to fit in a varchar(255) field.

When requesting other resources the access token must be passed via the Authorization header using the Bearer scheme *(e.g. Authorization: Bearer T9UNRV4sC9vr7ga)*.


## 2.3 Team, Project, Domain Services ##

#### *2.3.1 Team* ####

[team.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/team.json)

Services:

    GET, POST /V0.99/teams
- GET - Retrieve a list of teams where the currently logged on user is assigne to with his specific role
- Post - Add a new team

Long Url: - 

    GET, PUT, DELETE /V0.99/teams/{guid}


- GET - Retrieve a specific team
- PUT - Modify a specific team
- DELETE - Delete a specific team

Long Url: -

#### *2.3.2 Project* ####

[project.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/project.json), [extensions.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/extensions.json)

Services:

    GET, POST /V0.99/ teams/{guid}/projects

- GET - Retrieve a list of projects where the currently logged on user is assigned to with his specific roles.
- POST - Add a new project

Long Url: -

    GET, PUT, DELETE /V0.99/projects/{guid}

- GET - Retrieve a specific project
- PUT - Modify a specific project
- DELETE - Delete a specific project

Long URL: /V0.99/teams/{guid}/projects/{guid}

    GET, POST, PUT, DELETE /V0.99/projects/{guid}/thumbnail


- GET - Retrieve the project thumbnail (blob)
- POST - Upload the project thumbnail (blob)
- DELETE – Delete the project thumbnail

Long URL:  /V0.99/teams/{guid}/projects/{guid}/thumbnail


    GET, POST, PUT, DELETE /V0.99/projects/{guid}/extension

- GET - Retrieve the project extension schema
- POST - Add the project extension schema
- PUT - Change the project extension schema
- DELETE – Delete the project extension schema

Long URL:  /V0.99/teams/{guid}/projects/{guid}/extensions


#### *2.3.3 Domain* ####
[domain.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/domain.json)

Services:

    GET, POST /V0.99/projects/{guid}/domains

- GET - Retrieve a list of domains within a project where the currently logged on user is assigned to with his specific roles
- POST - Add a new domain to a project

Long URL: /V0.99/ teams/{ guid }/projects/{guid}/domains

    GET, PUT, DELETE /V0.99/domains/{guid}

- GET - Retrieve a specific domain
- PUT - Change a specific domain
- DELETE – Delete a specific domain

Long URL: /V0.99/ teams/{ guid }/projects/{guid}/domains/{guid}


## 2.4 Revision Services ##

[revision.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/revision.json)

Services:

    GET, POST /V0.99/domains/{guid}/revisions

- GET - Retrieve revisions of a domain 
- POST - Add a new revision to a domain; Multipart -> all informations retrieved from header

Long URL: /V0.99/ teams/{guid }/projects/{guid}/domains/{guid}/revisions


    GET, DELETE /V0.99/revisions/{guid}

- GET - Download a specific revision
- DELETE - Delete a specific revision

Long URL: /V0.99/ teams/{guid }/projects/{guid}/domains/{guid}/revisions/{guid}


## 2.5 BCF Services ##

#### *2.5.1 Topic* ####
[topic.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/topic.json), [revision.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/revision.json)

Services:

    GET, POST /V0.99/projects/{guid}/topics

- GET - Retrieve topics of a project (default sort = CreationDate)
- POST - Add a new topic to a project

Long URL: /V0.99/teams/{guid}/projects/{guid}/topics

    GET, PUT, DELETE /V0.99/topics/{guid}

- GET - Retrieve a specific topic
- PUT - Update a specific topic
- DELETE - Delete a specific topic

Long URL: /V0.99/teams/{guid}/projects/{guid}/topics/{guid}

    GET /V0.99/topics/{guid}/revisions

- GET - Retrieve all revisions related to a topic
- POST - Assign a revision to a topic

Long URL: /V0.99/teams/{guid}/projects/{guid}/topics/{guid}/revisions

    DELETE /V0.99/topics/{guid}/revisions/{guid}

- DELETE - Delete a revison to topic assignment

Long URL: /V0.99/teams/{guid}/projects/{guid}/topics/{guid}/revisions/{guid}


#### *2.5.2 Comment* ####
 [comment.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/comment.json)

Services:

    GET, POST /V0.99/topics/{guid}/comments

- GET - Retrieve comments of a topic
- POST - Add a new comment to a topic

Long URL: /V0.99/teams/{guid}/projects/{guid}/topics/{guid}/comments

    GET, PUT, DELETE /V0.99/comments/{guid}

- GET - Retrieve a specific comment
- PUT - Update a specific comment
- DELETE - Delete a specific comment

Long URL: /V0.99/teams/{guid}/projects/{guid}/topics/{guid}/comments/{guid}

    GET, POST, DELETE /V0.99/comments/{guid}/viewpoint

- GET - Retrieve the viewpoint assigned to a comment
- POST - Assign a viewpoint to a comment
- DELETE - Delete the viewpoint assigned to a comment

Long URL: /V0.99/teams/{guid}/projects/{guid}/topics/{guid}/comments/{guid}/viewpoint

    GET, POST, DELETE /V0.99/comments/{guid}/reply_to

- GET - Retrieve the replyTo comment related to a comment
- POST - Add a replyTo comment relation to a comment
- DELETE - Delete the replyTo comment relation on a comment

Long URL: /V0.99/teams/{guid}/projects/{guid}/topics/{guid}/comments/{guid}/reply_to

    GET, POST /V0.99/comments/{guid}/attachments

- GET - Retrieve the list of attachments on a comment
- POST - Add a new attachment to a comment

Long URL:  /V0.99/ teams/{guid}/projects/{guid}/topics/{guid}/comments/{guid}/attachments

    GET, PUT, DELETE /V0.99/attachments/{guid}

- GET - Retrieve a specific attachment
- PUT - Change a specific attachment
- DELETE – Delete a specific attachment

Long URL:  /V0.99/ teams/{guid}/projects/{guid}/topics/{guid}/comments/{guid}/attachments/{guid}

#### *2.5.3 Viewpoint* ####
 [viewpoint.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/viewpoint.json)

Services:

    GET, POST /V0.99/topics/{guid}/viewpoints


- GET - Retrieve viewpoints of a topic
- POST - Add a new viewpoint to a topic

Long URL: /V0.99/teams/{guid}/projects/{guid}/topics/{guid}/viewpoints

    GET, PUT, DELETE /V0.99/viewpoints/{guid}

- GET - Retrieve a specific viewpoint
- PUT - Modify a specific viewpoint
- DELETE – Delete a specific viewpoint

Long URL: /V0.99/teams/{guid}/projects/{guid}/topics/{guid}/viewpoints/{guid}


    GET, POST, DELETE /V0.99/viewpoints/{guid}/bitmap

- GET - Retrieve the bitmap related to a viewpoint
- PUT - Add a bitmap to the viewpoint
- DELETE – Delete the bitmap of the viewpoint

Long URL: /V0.99/teams/{guid}/projects/{guid}/topics/{guid}/viewpoints/{guid}/bitmap

#### *2.5.4 Component* ####
 [component.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/component.json)

Services:

    GET, POST /V0.99/viewpoints/{guid}/components

- GET - Retrieve components of a viewpoint
- POST - Add a new component to a viewpoint

Long URL: /V0.99/teams/{guid}/projects/{guid}/topics/{guid}/viewpoints/{guid}/components

    GET, PUT, DELETE /V0.99/components/{ifc_guid}

- GET - Retrieve a specific component
- PUT - Modify a specific component
- DELETE – Delete a specific component

Long URL: /V0.99/teams/{guid}/projects/{guid}/topics/{guid}/viewpoints/{guid}/components/{ifcguid}

#### *2.5.5 Related Topic* ####
 [related_topic.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/related_topic.json)

Services:

    GET, POST /V0.99/topics/{guid}/related_topics

- GET - Retrieve related topics to a topic
- POST - Add a new related_topic to a topic

Long URL: /V0.99/teams/{guid}/projects/{guid}/topics/{guid}/related_topics

    DELETE /V0.99/topics/{guid}/related_topics/{guid}

- DELETE – Delete a topic to topic relation

Long URL: /V0.99/teams/{guid}/projects/{guid}/topics/{guid}/related_topics/{guid}


#### *2.5.6 Document Reference* ####
 [document_reference.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/document_reference.json)

Services:

    GET, POST /V0.99/topics/{guid}/document_references

- GET - Retrieve documents referenced on a topic
- POST - Add a new document reference to a topic

Long URL: /V0.99/teams/{guid}/projects/{guid}/topics/{guid}/document_references

    DELETE /V0.99/topics/{guid}/document_references/{guid}

- GET - Retrieve a document referenced on a topic
- DELETE – Delete a document reference

Long URL: /V0.99/teams/{guid}/projects/{guid}/topics/{guid}/document_references/{guid} 

## 2.6 Attachment Services ##

[attachment.json](https://raw.githubusercontent.com/BuildingSMART/BCF-API/master/Schemas/attachment.json)

List of resources where a file can be attached:

- project
- domain
- revision
- comment
- viewpoint

Services:

    GET, POST /V0.99/...RESOURCE.../{guid}/attachments

- GET - Retrieve the list of attachments on a resource
- POST - Add a new attachment to a resource

Long URL:  /V0.99/...RESOURCE.../{guid}/...RESOURCE.../{guid}/...../attachments

    GET, PUT, DELETE /V0.99/attachments/{guid}

- GET - Retrieve a specific attachment
- PUT - Change a specific attachment
- DELETE – Delete a specific attachment

Long URL:  /V0.99/...RESOURCE.../{guid}/...RESOURCE.../{guid}/...../attachments/{guid}


## 2.7 User Services ##

#### *2.7.1 Registration* ####

To register a user, use the webplatform of the BIM – API server.

#### *2.7.2 Roles* ####

**Team-Owner:** Has all rights within a “Team and its “Projects” and “Domains”

**BIM-Manager:** Has all rights within a “Project” and its “Domains”

**Domain-Manager:** Has all rights within a “Domain”

**Domain-User:** Has specific rights within a “Domain”


#### *2.7.2 Rights* ####


<table border="1">
  <tr>
    <th></th>
    <th>Team-Owner</th>
    <th>BIM-Manager</th>
    <th>Domain-Manager</th>
    <th>Domain-User</th>
  </tr>
  <tr>
    <td>assign, remove team user</td>
    <td align="center">X</td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>create, edit, delete project</td>
    <td align="center">X</td>
    <td align="center">X</td>
    <td align="center"></td>
    <td></td>
  </tr>
  <tr>
    <td>invite user to project</td>
    <td align="center">X</td>
    <td align="center">X</td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>view project (attachments)</td>
    <td align="center">X</td>
    <td align="center">X</td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>create, edit, delete domain</td>
    <td align="center">X</td>
    <td align="center">X</td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>view all domains</td>
    <td align="center">X</td>
    <td align="center">X</td>
    <td></td>
    <td></td>
  </tr>
 <tr>
    <td>assign user to domain</td>
    <td align="center">X</td>
    <td align="center">X</td>
    <td></td>
    <td></td>
  </tr>
 <tr>
    <td>upload revision to domain</td>
    <td align="center">X</td>
    <td align="center">X</td>
    <td align="center">X</td>
    <td></td>
  </tr>
 <tr>
    <td>view domain</td>
    <td align="center">X</td>
    <td align="center">X</td>
    <td align="center">X</td>
    <td align="center">X</td>
  </tr>
 <tr>
    <td>add BCF</td>
    <td align="center">X</td>
    <td align="center">X</td>
    <td align="center">X</td>
    <td align="center">X</td>
  </tr>
 <tr>
    <td>edit BCF</td>
    <td align="center">X</td>
    <td align="center">Author</td>
    <td align="center">Author</td>
    <td align="center">Author</td>
  </tr>
 <tr>
    <td>delete BCF</td>
    <td align="center">X</td>
    <td align="center">Author & only last comment</td>
    <td align="center">Author & only last comment</td>
    <td align="center">Author & only last comment</td>
  </tr>
</table>

#### *2.7.3 Membership* ####

A user can be assigned to a *Team*, *Project* or *Domain*.

    GET, POST /V0.99/objects/{guid}/users

- GET - Retrieve the list of users assigned to an object with their specific roles
- POST - Assign a user with its role to an object

Long Url: -

    PUT, DELETE /V0.99/objects/{guid}/users/{user_id}

- PUT - Change a user role on an object
- DELETE - Remove a user from an object

Long Url: -

    GET /V0.99/users/{user_id}

- GET - Retrieve a list of objects wher a user is assigned to with his specific roles

Long Url: - 


----------


