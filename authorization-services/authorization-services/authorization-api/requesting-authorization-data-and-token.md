# Requesting Authorization Data and Token

Client applications using the UMA protocol can use a specific endpoint to obtain a special security token called a requesting party token (RPT). This token consists of all the permissions granted to a user as a result of the evaluation of the permissions and authorization policies associated with the resources being requested. With an RPT, client applications can gain access to protected resources at the resource server.

```bash
http://${host}:${port}/auth/realms/${realm_name}/authz/authorize
```

When requesting an RPT, you need to provide two things:

* A [permission ticket](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/service/protection/permission-api-papi.html#\_service\_protection\_permission\_api\_papi) with the resources you want to access
* The [authorization API token (AAT)](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/service/authorization/whatis-obtain-aat.html#\_service\_authorization\_aat) (as a bearer token) representing a user’s identity and his consent to access authorization data on his behalf.

```bash
curl -X POST
    -H "Authorization: Bearer ${AAT}" -d '{
    "ticket" : ${PERMISSION_TICKET}
}' "http://localhost:8080/auth/realms/hello-world-authz/authz/authorize"
```

As a result, the server response is:

```json
{"rpt":"${RPT}"}
```

**Requesting Party Token**

A Requesting Party Token (RPT) is a [JSON web token (JWT)](https://tools.ietf.org/html/rfc7519) digitally signed using [JSON Web Signature (JWS)](https://www.rfc-editor.org/rfc/rfc7515.txt). The token is built based on the AAT sent by the client during the authorization process.

When you decode an RPT you will see something like:

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
