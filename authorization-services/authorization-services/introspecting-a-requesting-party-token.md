# Introspecting a Requesting Party Token

Sometimes you might want to introspect a requesting party token (RPT) to check its validity or obtain the permissions within the token to enforce authorization decisions on the resource server side.

There are two main use cases where token introspection can help you:

* When client applications need to query the token validity to obtain a new one with the same or additional permissions
* When enforcing authorization decisions at the resource server side, especially when none of the built-in [policy enforcers](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/enforcer/overview.html#\_enforcer\_overview) fits your application

**Obtaining Information about an RPT**

The token introspection is essentially a [OAuth2 token introspection](https://tools.ietf.org/html/rfc7662)-compliant endpoint from which you can obtain information about an RPT.

```bash
http://${host}:${port}/auth/realms/${realm_name}/protocol/openid-connect/token/introspect
```

To introspect an RPT using this endpoint, you can send a request to the server as follows:

```bash
curl -X POST \
    -H "Authorization: Basic aGVsbG8td29ybGQtYXV0aHotc2VydmljZTpzZWNyZXQ=" \
    -H "Content-Type: application/x-www-form-urlencoded" \
    -d 'token_type_hint=requesting_party_token&token=${RPT}' \
    "http://localhost:8080/auth/realms/hello-world-authz/protocol/openid-connect/token/introspect"
```

| Note | The request above is using HTTP BASIC and passing the client’s credentials (client ID and secret) to authenticate the client attempting to introspect the token, but you can use any other client authentication method supported by Keycloak. |
| ---- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

The introspection endpoint expects two parameters:

*   **token\_type\_hint**

    Use **requesting\_party\_token** as the value for this parameter, which indicates that you want to introspect an RPT.
*   **token**

    Use the token string as it was returned by the server during the authorization process as the value for this parameter.

As a result, the server response is:

```json
{
  "permissions": [
    {
      "resource_set_id": "90ccc6fc-b296-4cd1-881e-089e1ee15957",
      "resource_set_name": "Hello World Resource"
    }
  ],
  "exp": 1465314139,
  "nbf": 0,
  "iat": 1465313839,
  "aud": "hello-world-authz-service",
  "active": true
}
```

If the RPT is not active, this response is returned instead:

```json
{
  "active": false
}
```

**Do I Need to Invoke the Server Every Time I Want to Introspect an RPT?**

No. Both [Authorization](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/service/authorization/authorization-api.html#\_service\_authorization\_api) and [Entitlement](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/service/entitlement/entitlement-api.html#\_service\_entitlement\_api) APIs use the [JSON web token (JWT)](https://tools.ietf.org/html/rfc7519) specification as the default format for RPTs.

If you want to validate these tokens without a call to the remote introspection endpoint, you can decode the RPT and query for its validity locally. Once you decode the token, you can also use the permissions within the token to enforce authorization decisions.

This is essentially what the [policy enforcers](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/enforcer/overview.html#\_enforcer\_overview) do. Be sure to:

* Validate the signature of the RPT (based on the realm’s public key)
* Query for token validity based on its _exp_, _iat_, and _aud_ claims
