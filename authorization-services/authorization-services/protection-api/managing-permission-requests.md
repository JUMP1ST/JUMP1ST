# Managing Permission Requests

Resource servers using the UMA protocol can use a specific endpoint to manage permission requests. This endpoint provides a UMA-compliant flow for registering permission requests and obtaining a permission ticket.

```bash
http://${host}:${port}/auth/realms/${realm_name}/authz/protection/permission
```

A [permission ticket](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/overview/terminology.html#\_overview\_terminology\_permission\_ticket) is a special security token type representing a permission request. Per the UMA specification, a permission ticket is:

`A correlation handle that is conveyed from an authorization server to a resource server, from a resource server to a client, and ultimately from a client back to an authorization server, to enable the authorization server to assess the correct policies to apply to a request for authorization data.`

| Note | _Permission ticket support is limited_. In the full UMA protocol, resource servers can register permission requests in the server to support authorization flows where a resource owner (the user that owns a resource being requested) can approve access to his resources by third parties, among other ways. This represents one of the main features of the UMA specification: resource owners can control their own resources and the policies that govern them. Currently Keycloak UMA implementation support is very limited in this regard. For example, the system does not store permission tickets on the server and we are essentially using UMA to provide API security and base our authorization offerings. In the future, full support of UMA and other use cases is planned. |
| ---- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

In most cases, you won’t need to deal with this endpoint directly. Keycloak provides a [policy enforcer](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/enforcer/overview.html#\_enforcer\_overview) that enables UMA for your resource server so it can obtain a permission ticket from the authorization server, return this ticket to client application, and enforce authorization decisions based on a final requesting party token (RPT).
