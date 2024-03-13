# Unspecific Redirect URIs

For the [Authorization Code Flow](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/sso-protocols/oidc.html#\_oidc-auth-flows), if you register redirect URIs that are too general, then it would be possible for a rogue client to impersonate a different client that has a broader scope of access. This could happen for instance if two clients live under the same domain. So, it’s a good idea to make your registered redirect URIs as specific as feasible.
