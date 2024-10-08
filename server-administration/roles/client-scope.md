# Client Scope

When an OIDC access token or SAML assertion is created, all the user role mappings of the user are, by default, added as claims within the token or assertion. Applications use this information to make access decisions on the resources controlled by that application. In Keycloak, access tokens are digitally signed and can actually be re-used by the application to invoke on other remotely secured REST services. This means that if an application gets compromised or there is a rogue client registered with the realm, attackers can get access tokens that have a broad range of permissions and your whole network is compromised. This is where _client scope_ becomes important.

_Client scope_ is a way to limit the roles that get declared inside an access token. When a client requests that a user be authenticated, the access token they receive back will only contain the role mappings you’ve explicitly specified for the client’s scope. This allows you to limit the permissions each individual access token has rather than giving the client access to all of the user’s permissions. By default, each client gets all the role mappings of the user. You can view this in the `Scope` tab of each client.

Full Scope

![full-client-scope.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/full-client-scope.png)

You can see from the picture that the effective roles of the scope are every declared role in the realm. To change this default behavior, you must explicitly turn off the `Full Scope Allowed` switch and declare the specific roles you want in each individual client. Alternatively, you can also use [client templates](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/clients/clienttemplates.html#\_client\_templates) to define the scope for a whole set of clients.

Partial Scope

![client-scope.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/client-scope.png)
