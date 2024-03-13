# Limiting Scope

By default, each new client application has an unlimited scope. This means that every access token that is created for that client will contain all the permissions the user has. If the client gets compromised and the access token is leaked, then each system that the user has permission to access is now also compromised. It is highly suggested that you limit the roles an access token is assigned by using the [Scope menu](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/roles/client-scope.html#\_client\_scope) for each client.
