# Available User Session Data

After a user logs in from the external IDP, there’s some additional user session note data that Keycloak stores that you can access. This data can be propagated to the client requesting a login via the token or SAML assertion being passed back to it by using an appropriate client mapper.

identity\_provider

This is the IDP alias of the broker used to perform the login.

identity\_provider\_identity

This is the IDP username of the currently authenticated user. This is often same like the Keycloak username, but doesn’t necessarily needs to be. For example Keycloak user `john` can be linked to the Facebook user [`[email protected]`](https://wjw465150.gitbooks.io/cdn-cgi/l/email-protection), so in that case value of user session note will be [`[email protected]`](https://wjw465150.gitbooks.io/cdn-cgi/l/email-protection) .

You can use a [Protocol Mapper](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/clients/protocol-mappers.html#\_protocol-mappers) of type `User Session Note` to propagate this information to your clients.
