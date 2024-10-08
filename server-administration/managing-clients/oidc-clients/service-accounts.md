# Service Accounts

Each OIDC client has a built-in _service account_ which allows it to obtain an access token. This is covered in the OAuth 2.0 specifiation under [Client Credentials Grant](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/sso-protocols/oidc.html#\_client\_credentials\_grant). To use this feature you must set the [Access Type](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/clients/client-oidc.html#\_access-type) of your client to `confidential`. When you do this, the `Service Accounts Enabled` switch will appear. You need to turn on this switch. Also make sure that you have configured your [client credentials](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/clients/oidc/confidential.html#\_client-credentials).

To use it you must have registered a valid `confidential` Client and you need to check the switch `Service Accounts Enabled` in Keycloak admin console for this client. In tab `Service Account Roles` you can configure the roles available to the service account retrieved on behalf of this client. Don’t forget that you need those roles to be available in Scopes of this client as well (unless you have `Full Scope Allowed` on). As in normal login, roles from access token are the intersection of scopes and the service account roles.

The REST URL to invoke on is `/{server-root-usualy-auth}/realms/{realm-name}/protocol/openid-connect/token`. Invoking on this URL is a POST request and requires you to post the client credentials. By default, client credentials are represented by clientId and clientSecret of the client in `Authorization: Basic` header, but you can also authenticate the client with a signed JWT assertion or any other custom mechanism for client authentication. You also need to use the parameter `grant_type=client_credentials` as per the OAuth2 specification.

For example the POST invocation to retrieve a service account can look like this:

```
    POST /auth/realms/demo/protocol/openid-connect/token
    Authorization: Basic cHJvZHVjdC1zYS1jbGllbnQ6cGFzc3dvcmQ=
    Content-Type: application/x-www-form-urlencoded

    grant_type=client_credentials
```

The response would be this [standard JSON document](http://tools.ietf.org/html/rfc6749#section-4.4.3) from the OAuth 2.0 specification.

```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
    "access_token":"2YotnFZFEjr1zCsicMWpAA",
    "token_type":"bearer",
    "expires_in":60,
    "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
    "refresh_expires_in":600,
    "id_token":"tGzv3JOkF0XG5Qx2TlKWIA",
    "not-before-policy":0,
    "session_state":"234234-234234-234234"
}
```

The retrieved access token can be refreshed or logged out by an out-of-bound request.
