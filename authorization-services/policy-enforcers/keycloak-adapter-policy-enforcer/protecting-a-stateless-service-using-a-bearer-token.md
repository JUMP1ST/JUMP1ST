# Protecting a Stateless Service Using a Bearer Token

If the adapter is configured with the `bearer-only` configuration option, the policy enforcer decides whether a request to access a protected resource is allowed or denied based on the permissions of the bearer token.

1. HTTP GET example passing an RPT as a bearer token

```bash
GET /my-resource-server/my-protected-resource HTTP/1.1
Host: host.com
Authorization: Bearer ${RPT}
...
```

In this example, a **keycloak.json** file in your application is similar to the following:

Example of WEB-INF/keycloak.json with the bearer-only configuration option

```json
...
"bearer-only" : true,
...
```

**Authorization Response**

When a client tries to access a resource server with a bearer token that is lacking permissions to access a protected resource, the resource server responds with a **401** status code and a `WWW-Authenticate` header. The value of the `WWW-Authenticate` header depends on the authorization protocol in use by the resource server.

Here is an example of a response from a resource server that is using UMA as the authorization protocol:

```bash
HTTP/1.1 401 Unauthorized
WWW-Authenticate: UMA realm="photoz-restful-api",as_uri="http://localhost:8080/auth/realms/photoz/authz/authorize",ticket="${PERMISSION_TICKET}"
```

And another example when the resource server is using the Entitlement protocol:

```bash
HTTP/1.1 401 Unauthorized
WWW-Authenticate: KC_ETT realm="photoz-restful-api",as_uri="http://localhost:8080/auth/realms/photoz/authz/entitlement"
```

Once a client receives a response from the server, it examines the status code and `WWW-Authenticate` header to obtain an RPT from the Keycloak Server.

\
