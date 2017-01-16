# BCF API OAuth2 Example

This example describes an example OAuth2 workflow using the _Authorization Code Grant_ flow as per [section 4.1 of the OAuth2 specification](https://tools.ietf.org/html/rfc6749#section-4.1). Uris of required endpoints are assumed to have been obtained from the authentication resource as described in [section 3.2 of the BCF API specification](https://github.com/BuildingSMART/BCF-API#321-obtaining-authentication-information).

For this example, it is assumed that the client has been registered with the server in advance and has been issued valid credentials in the form of `client_id` and `client_secret`.

## Client Request

To initiate the workflow, the client sends the user to the **"oauth2\_auth_url"** with the following parameters added:

|parameter|value|
|-------------|------|
|response_type|`code` as string literal|
|client_id|your client id|
|state|unique user defined value|

Example URL:

    GET https://example.com/bcf/oauth2/auth?response_type=code&client_id=<your_client_id>&state=<user_defined_string>

_On Windows operating systems, it is possible to open the systems default browser by using the url to start a new process._

Example redirected URL:

    https://YourWebsite.com/retrieveCode?code=<server_generated_code>&state=<user_defined_string>

The BCF API server will ask the user to confirm that the client may access resources on his behalf. On authorization, the server redirects to an url that has been defined by the client author in advance. The generated `code` parameter will be appended as query parameter. Additionally, the `state` parameter is included in the redirection, it may be used to match server responses to client requests issued by your application.

If the user denies client access, there will be an `error` query parameter in the redirection indicating an error reason as described in [section 4.1.2.1 of the OAuth2 specification](https://tools.ietf.org/html/rfc6749#section-4.1.2.1).

## Token Request

With the obtained _authorization code_, the client is able to request an access token from the server. The  **"oauth2\_token_url"** from the authentication resource is used to send token requests to, for example:

    POST https://example.com/bcf/oauth2/token

**Parameters**

|parameter|type|description|
|---------|----|-----------|
|access_token|string|The issued OAuth2 token|
|token_type|string|Always `bearer`|
|expires_in|integer|The lifetime of the access token in seconds|
|refresh_token|string|The issued OAuth2 refresh token, one-time-usable only|

The POST request should be done via HTTP Basic Authorization with your applications `client_id` as the username and your `client_secret` as the password.

**Example Request**

    POST https://example.com/bcf/oauth2/token?grant_type=authorization_code&code=<your_access_code>

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

## Refresh Token Request

The process to retrieve a refresh token is exactly the same as retrieving a token via the code workflow except the `refresh_token` is sent instead of the `code` parameter and the `refresh_token` grant type is used.

**Example Request**

    POST https://example.com/bcf/oauth2/token?grant_type=refresh_token&refresh_token=<your_refresh_token>

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

## Requesting Resources

When requesting other resources the access token must be passed via the `Authorization` header using the `Bearer` scheme (e.g. `Authorization: Bearer Zjk1YjYyNDQtOTgwMy0xMWU0LWIxMDAtMTIzYjkzZjc1Y2Jh`).