# Client Suggested Identity Provider

OIDC applications can bypass the Keycloak login page by specifying a hint on which identity provider they want to use.

This is done by setting the `kc_idp_hint` query parameter in the Authorization Code Flow authorization endpoint.

Keycloak OIDC client adapters also allow you to specify this query parameter when you access a secured resource at the application.

For example

```java
GET /myapplication.com?kc_idp_hint=facebook HTTP/1.1
Host: localhost:8080
```

In this case, is expected that your realm has an identity provider with an alias `facebook`. If this provider doesn’t exist the login form will be displayed.

If you are using `keycloak.js` adapter, you can also achieve the same behavior:

```java
var keycloak = new Keycloak('keycloak.json');

keycloak.createLoginUrl({
	idpHint: 'facebook'
});
```

The `kc_idp_hint` query parameter also allows the client to override the default identity provider if one is configured for the `Identity Provider Redirector` authenticator. The client can also disable the automatic redirecting by setting the `kc_idp_hint` query parameter to an empty value.
