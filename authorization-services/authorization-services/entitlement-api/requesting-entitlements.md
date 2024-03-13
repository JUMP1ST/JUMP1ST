# Requesting Entitlements

Client applications can use a specific endpoint to obtain a special security token called a requesting party token (RPT). This token consists of all the entitlements (or permissions) for a user as a result of the evaluation of the permissions and authorization policies associated with the resources being requested. With an RPT, client applications can gain access to protected resources at the resource server.

```bash
http://${host}:${port}/auth/realms/${realm_name}/authz/entitlement
```

**Obtaining Entitlements**

The easiest way to obtain entitlements for a specific user is using an HTTP GET request. For example, using curl:

```bash
curl -X GET \
    -H "Authorization: Bearer ${access_token}" \
    "http://localhost:8080/auth/realms/hello-world-authz/authz/entitlement/${resource_server_id}"
```

| Note | When requesting entitlements using this endpoint, you must provide the access\_token (as a bearer token) representing a user’s identity and his consent to access authorization data on his behalf. |
| ---- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

In the curl example, **${resource\_server\_id}** is the **client\_id** registered with the client application acting as a resource server.

As a result, the server response is:

```json
{
  "rpt": ${RPT}
}
```

Using this method to obtain entitlements, the server responds to the requesting client with **all** entitlements for a user, based on the evaluation of the permissions and authorization policies associated with the resources managed by the resource server.

**Obtaining Entitlements for a Specific Set of Resources**

You can also use the entitlements endpoint to obtain a user’s entitlements for a set of one or more resources. For example, using curl:

```bash
curl -X POST -H "Authorization: Bearer ${access_token}" -d '{
    "permissions" : [
        {
            "resource_set_name" : "Hello World Resource"
        }
    ]
}' "http://localhost:8080/auth/realms/hello-world-authz/authz/entitlement/hello-world-authz-service"
```

As a result, the server response is:

```json
{
  "rpt": ${RPT}
}
```

Unlike the GET version, the server responds with an RPT holding the permissions granted during the evaluation of the permissions and authorization policies associated with the resources being requested.

When requesting entitlements, you can also specify the scopes you want to access. For example, using curl:

```bash
curl -X POST -H "Authorization: Bearer ${access_token}" -d '{
    "permissions" : [
        {
            "resource_set_name" : "Hello World Resource",
            "scopes" : [
                "urn:my-app.com:scopes:view"
            ]
        }
    ]
}' "http://localhost:8080/auth/realms/hello-world-authz/authz/entitlement/hello-world-authz-service"
```

**Requesting Party Token**

A requesting party token (RPT) is a [JSON web token (JWT)](https://tools.ietf.org/html/rfc7519) digitally signed using [JSON web signature (JWS)](https://www.rfc-editor.org/rfc/rfc7515.txt). The token is built based on the access\_token sent by the client during the authorization process.

When you decode an RPT, you see a payload similar to the following:

```json
{
  "authorization": {
      "permissions": [
        {
          "resource_set_id": "d2fe9843-6462-4bfc-baba-b5787bb6e0e7",
          "resource_set_name": "Hello World Resource"
        }
      ]
  },
  "jti": "d6109a09-78fd-4998-bf89-95730dfd0892-1464906679405",
  "exp": 1464906971,
  "nbf": 0,
  "iat": 1464906671,
  "sub": "f1888f4d-5172-4359-be0c-af338505d86c",
  "typ": "kc_ett",
  "azp": "hello-world-authz-service"
}
```

From this token you can obtain all permissions granted by the server from the **permissions** claim.

\
