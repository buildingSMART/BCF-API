|        |        |    
| ------------- |-------------|
|/V0.99|GET
|/V0.99/schema|GET (topic.json)
|/V0.99/teams|GET, POST
|/V0.99/teams/{id}|GET, PUT, DELETE
|/V0.99/ teams/{id}/projects|GET, POST
|/V0.99/ teams/{id}/projects/{id}|GET, PUT, DELETE
|/V0.99/ teams/{id}/projects /{id}/schema|GET,POST, PUT, DELETE (extensions.json)
|/V0.99/ teams/{id}/projects/{id}/disciplines|GET, POST, DELETE
|/V0.99/ teams/{id}/projects/{id}/disciplines/{id}|GET, PUT, DELETE
|/V0.99/ teams/{id}/projects/{id}/disciplines/{id}/revisions|GET, POST
|/V0.99/ teams/{id}/projects/{id}/disciplines/{id}/revisions/{id}|GET, DELETE
|/V0.99/ teams/{id}/projects/{id}/disciplines/{id}/revisions/{id}/{reference}|GET
|
|/V0.99/teams/{id}/projects/{id}/topics|GET, POST
|/V0.99/teams/{id}/projects/{id}/topics/{guid}|GET, PUT, DELETE
|/V0.99/teams/{id}/projects/{id}/topics/{guid}/header|GET, POST
|/V0.99/ teams/{id}/projects/{id}/topics/{guid}/document_references|GET, POST
|/V0.99/teams/{id}/projects/{id}/topics/{guid}/document_references/{guid}|GET, DELETE
|
|/V0.99/ teams/{id}/projects/{id}/topics/{guid}/bim_snippet|GET, POST, (DELETE)
|/V0.99/ teams/{id}/projects/{id}/topics/{guid}/bim_snippet/{reference}|GET
|/V0.99/ teams/{id}/projects/{id}/topics/{guid}/bim_snippet/{schema}|GET
|
|/V0.99/ teams/{id}/projects/{id}/topics/{guid}/related_topics|GET, POST
|/V0.99/ teams/{id}/projects/{id}/topics/{guid}/related_topics/{guid}|(DELETE)
|
|/V0.99/ teams/{id}/projects/{id}/topics/{guid}/comments|GET, POST
|/V0.99/ teams/{id}/projects/{id}/topics/{guid}/comments/{guid}|GET, PUT, (DELETE)
|
|/V0.99/ teams/{id}/projects/{id}/topics/{guid}|/Viewpoints|GET, POST
|/V0.99/ teams/{id}/projects/{id}/topics/{guid}/ viewpoints /{guid}|GET, PUT, (DELETE)
|
|/V0.99/ teams/{id}/projects/{id}/topics/{guid}|/Visinfo|GET, POST, (PUT), (DELETE)
|




