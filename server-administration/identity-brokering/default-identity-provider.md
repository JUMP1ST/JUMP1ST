# Default Identity Provider

It’s possible to automatically redirect to a identity provider instead of displaying the login form. To enable this go to `Authentication` select the `Browser` flow. Then click on config for the `Identity Provider Redirector` authenticator. Set `Default Identity Provider` to the alias of the identity provider you want to automatically redirect users to.

If the configured default identity provider is not found the login form will be displayed instead.

This authenticator is also responsible for dealing with the `kc_idp_hint` query parameter. See [client suggested identity provider](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/identity-broker/suggested.html#\_client\_suggested\_idp) section for more details.
