# Other OpenID Connect Libraries

Keycloak can be secured by supplied adapters that are usually easier to use and provide better integration with Keycloak. However, if an adapter is not available for your programming language, framework, or platform you might opt to use a generic OpenID Connect Resource Provider (RP) library instead. This chapter describes details specific to Keycloak and does not contain specific protocol details. For more information see the [OpenID Connect specifications](http://openid.net/connect/) and [OAuth2 specification](https://tools.ietf.org/html/rfc6749).

**Endpoints**

The most important endpoint to understand is the `well-known` configuration endpoint. It lists endpoints and other configuration options relevant to the OpenID Connect implementation in Keycloak. The endpoint is:

```
/realms/{realm-name}/.well-known/openid-configuration
```

To obtain the full URL, add the base URL for Keycloak and replace `{realm-name}` with the name of your realm. For example:

http://localhost:8080/auth/realms/master/.well-known/openid-configuration

Some RP libraries retrieve all required endpoints from this endpoint, but for others you might need to list the endpoints individually.

**Authorization Endpoint**

```
/realms/{realm-name}/protocol/openid-connect/auth
```

The authorization endpoint performs authentication of the end-user. This is done by redirecting the user agent to this endpoint.

For more details see the [Authorization Endpoint](http://openid.net/specs/openid-connect-core-1\_0.html#AuthorizationEndpoint) section in the OpenID Connect specification.

**Token Endpoint**

```
/realms/{realm-name}/protocol/openid-connect/token
```

The token endpoint is used to obtain tokens. Tokens can either be obtained by exchanging an authorization code or by supplying credentials directly depending on what flow is used. The token endpoint is also used to obtain new access tokens when they expire.

For more details see the [Token Endpoint](http://openid.net/specs/openid-connect-core-1\_0.html#TokenEndpoint) section in the OpenID Connect specification.

**Userinfo Endpoint**

```
/realms/{realm-name}/protocol/openid-connect/userinfo
```

The userinfo endpoint returns standard claims about the authenticated user, and is protected by a bearer token.

For more details see the [Userinfo Endpoint](http://openid.net/specs/openid-connect-core-1\_0.html#UserInfo) section in the OpenID Connect specification.

**Logout Endpoint**

```
/realms/{realm-name}/protocol/openid-connect/logout
```

The logout endpoint logs out the authenticated user.

The user agent can be redirected to the endpoint, in which case the active user session is logged out. Afterward the user agent is redirected back to the application.

The endpoint can also be invoked directly by the application. To invoke this endpoint directly the refresh token needs to be included as well as the credentials required to authenticate the client.

**Certificate Endpoint**

```
/realms/{realm-name}/protocol/openid-connect/certs
```

The certificate endpoint returns the public keys enabled by the realm, encoded as a JSON Web Key (JWK). Depending on the realm settings there can be one or more keys enabled for verifying tokens. For more information see the [Server Administration](https://keycloak.gitbooks.io/documentation/content/server\_admin/index.html) and the [JSON Web Key specification](https://tools.ietf.org/html/rfc7517).

**Introspection Endpoint**

```
/realms/{realm-name}/protocol/openid-connect/token/introspect
```

The introspection endpoint is used to retrieve the active state of a token. It is can only be invoked by confidential clients.

For more details see [OAuth 2.0 Token Introspection specification](https://tools.ietf.org/html/rfc7662).

**Dynamic Client Registration Endpoint**

```
/realms/{realm-name}/clients-registrations/openid-connect
```

The dynamic client registration endpoint is used to dynamically register clients.

For more details see the [Client Registration chapter](https://wjw465150.gitbooks.io/keycloak-documentation/content/securing\_apps/topics/client-registration.html#\_client\_registration) and the [OpenID Connect Dynamic Client Registration specification](https://openid.net/specs/openid-connect-registration-1\_0.html).

**Flows**

**Authorization Code**

The Authorization Code flow redirects the user agent to Keycloak. Once the user has successfully authenticated with Keycloak an Authorization Code is created and the user agent is redirected back to the application. The application then uses the authorization code along with its credentials to obtain an Access Token, Refresh Token and ID Token from Keycloak.

The flow is targeted towards web applications, but is also recommended for native applications, including mobile applications, where it is possible to embed a user agent.

For more details refer to the [Authorization Code Flow](http://openid.net/specs/openid-connect-core-1\_0.html#CodeFlowAuth) in the OpenID Connect specification.

**Implicit**

The Implicit flow redirects works similarly to the Authorization Code flow, but instead of returning a Authorization Code the Access Token and ID Token is returned. This reduces the need for the extra invocation to exchange the Authorization Code for an Access Token. However, it does not include a Refresh Token. This results in the need to either permit Access Tokens with a long expiration, which is problematic as it’s very hard to invalidate these. Or requires a new redirect to obtain new Access Token once the initial Access Token has expired. The Implicit flow is useful if the application only wants to authenticate the user and deals with logout itself.

There’s also a Hybrid flow where both the Access Token and an Authorization Code is returned.

One thing to note is that both the Implicit flow and Hybrid flow has potential security risks as the Access Token may be leaked through web server logs and browser history. This is somewhat mitigated by using short expiration for Access Tokens.

For more details refer to the [Implicit Flow](http://openid.net/specs/openid-connect-core-1\_0.html#ImplicitFlowAuth) in the OpenID Connect specification.

**Resource Owner Password Credentials**

Resource Owner Password Credentials, referred to as Direct Grant in Keycloak, allows exchanging user credentials for tokens. It’s not recommended to use this flow unless you absolutely need to. Examples where this could be useful are legacy applications and command-line interfaces.

There are a number of limitations of using this flow, including:

* User credentials are exposed to the application
* Applications need login pages
* Application needs to be aware of the authentication scheme
* Changes to authentication flow requires changes to application
* No support for identity brokering or social login
* Flows are not supported (user self-registration, required actions, etc.)

For a client to be permitted to use the Resource Owner Password Credentials grant the client has to have the `Direct Access Grants Enabled` option enabled.

This flow is not included in OpenID Connect, but is a part of the OAuth 2.0 specification.

For more details refer to the [Resource Owner Password Credentials Grant](https://tools.ietf.org/html/rfc6749#section-4.3) chapter in the OAuth 2.0 specification.

**Example using CURL**

The following example shows how to obtain an access token for a user in the realm `master` with username `user` and password `password`. The example is using the confidential client `myclient`:

```bash
curl \
  -d "client_id=myclient" \
  -d "client_secret=40cc097b-2a57-4c17-b36a-8fdf3fc2d578" \
  -d "username=user" \
  -d "password=password" \
  -d "grant_type=password" \
  "http://localhost:8080/auth/realms/master/protocol/openid-connect/token"
```

**Client Credentials**

Client Credentials is used when clients (applications and services) wants to obtain access on behalf of themselves rather than on behalf of a user. This can for example be useful for background services that applies changes to the system in general rather than for a specific user.

Keycloak provides support for clients to authenticate either with a secret or with public/private keys.

This flow is not included in OpenID Connect, but is a part of the OAuth 2.0 specification.

For more details refer to the [Client Credentials Grant](https://tools.ietf.org/html/rfc6749#section-4.4) chapter in the OAuth 2.0 specification.

**Redirect URIs**

When using the redirect based flows it’s important to use valid redirect uris for your clients. The redirect uris should be as specific as possible. This especially applies to client-side (public clients) applications. Failing to do so could result in:

* Open redirects - this can allow attackers to create spoof links that looks like they are coming from your domain
* Unauthorized entry - when users are already authenticated with Keycloak an attacker can use a public client where redirect uris have not be configured correctly to gain access by redirecting the user without the users knowledge

In production for web applications always use `https` for all redirect URIs. Do not allow redirects to http.

There’s also a few special redirect URIs:

`http://localhost`

This redirect URI is useful for native applications and allows the native application to create a web server on a random port that can be used to obtain the authorization code. This redirect uri allows any port.

`urn:ietf:wg:oauth:2.0:oob`

If its not possible to start a web server in the client (or a browser is not available) it is possible to use the special `urn:ietf:wg:oauth:2.0:oob` redirect uri. When this redirect uri is used Keycloak displays a page with the code in the title and in a box on the page. The application can either detect that the browser title has changed, or the user can copy/paste the code manually to the application. With this redirect uri it is also possible for a user to use a different device to obtain a code to paste back to the application.

\
