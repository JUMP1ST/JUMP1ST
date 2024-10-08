# Open ID Connect

[Open ID Connect](http://openid.net/connect/) (OIDC) is an authentication protocol that is an extension of [OAuth 2.0](https://tools.ietf.org/html/rfc6749). While OAuth 2.0 is only a framework for building authorization protocols and is mainly incomplete, OIDC is a full-fledged authentication and authorization protocol. OIDC also makes heavy use of the [Json Web Token](https://jwt.io/) (JWT) set of standards. These standards define an identity token JSON format and ways to digitally sign and encrypt that data in a compact and web-friendly way.

There are really two types of use cases when using OIDC. The first is an application that asks the Keycloak server to authenticate a user for them. After a successful login, the application will receive an _identity token_ and an _access token_. The _identity token_ contains information about the user such as username, email, and other profile information. The _access token_ is digitally signed by the realm and contains access information (like user role mappings) that the application can use to determine what resources the user is allowed to access on the application.

The second type of use cases is that of a client that wants to gain access to remote services. In this case, the client asks Keycloak to obtain an _access token_ it can use to invoke on other remote services on behalf of the user. Keycloak authenticates the user then asks the user for consent to grant access to the client requesting it. The client then receives the _access token_. This _access token_ is digitally signed by the realm. The client can make REST invocations on remote services using this _access token_. The REST service extracts the _access token_, verifies the signature of the token, then decides based on access information within the token whether or not to process the request.

**OIDC Auth Flows**

OIDC has different ways for a client or application to authenticate a user and receive an _identity_ and _access_ token. Which path you use depends greatly on the type of application or client requesting access. All of these flows are described in the OIDC and OAuth 2.0 specifications so only a brief overview will be provided here.

**Authorization Code Flow**

This is a browser-based protocol and it is what we recommend you use to authenticate and authorize browser-based applications. It makes heavy use of browser redirects to obtain an _identity_ and _access_ token. Here’s a brief summary:

1. Browser visits application. The application notices the user is not logged in, so it redirects the browser to Keycloak to be authenticated. The application passes along a callback URL (a redirect URL) as a query parameter in this browser redirect that Keycloak will use when it finishes authentication.
2. Keycloak authenticates the user and creates a one-time, very short lived, temporary code. Keycloak redirects back to the application using the callback URL provided earlier and additionally adds the temporary code as a query parameter in the callback URL.
3. The application extracts the temporary code and makes a background out of band REST invocation to Keycloak to exchange the code for an _identity_, _access_ and _refresh_ token. Once this temporary code has been used once to obtain the tokens, it can never be used again. This prevents potential replay attacks.

It is important to note that _access_ tokens are usually short lived and often expired after only minutes. The additional _refresh_ token that was transmitted by the login protocol allows the application to obtain a new access token after it expires. This refresh protocol is important in the situation of a compromised system. If access tokens are short lived, the whole system is only vulnerable to a stolen token for the lifetime of the access token. Future refresh token requests will fail if an admin has revoked access. This makes things more secure and more scalable.

Another important aspect of this flow is the concept of a _public_ vs. a _confidential_ client. _Confidential_ clients are required to provide a client secret when they exchange the temporary codes for tokens. _Public_ clients are not required to provide this client secret. _Public_ clients are perfectly fine so long as HTTPS is strictly enforced and you are very strict about what redirect URIs are registered for the client. HTML5/JavaScript clients always have to be _public_ clients because there is no way to transmit the client secret to them in a secure manner. Again, this is ok so long as you use HTTPS and strictly enforce redirect URI registration. This guide goes more detail into this in the [Managing Clients](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/clients.html#\_clients) chapter.

**Implicit Flow**

This is a browser-based protocol that is similar to Authorization Code Flow except there are fewer requests and no refresh tokens involved. We do not recommend this flow as there remains the possibility of _access_ tokens being leaked in the browser history as tokens are transmitted via redirect URIs (see below). Also, since this flow doesn’t provide the client with a refresh token, access tokens would either have to be long-lived or users would have to re-authenticate when they expired. This flow is supported because it is in the OIDC and OAuth 2.0 specification. Here’s a brief summary of the protocol:

1. Browser visits application. The application notices the user is not logged in, so it redirects the browser to Keycloak to be authenticated. The application passes along a callback URL (a redirect URL) as a query parameter in this browser redirect that Keycloak will use when it finishes authentication.
2. Keycloak authenticates the user and creates an _identity_ and _access_ token. Keycloak redirects back to the application using the callback URL provided earlier and additionally adding the _identity_ and _access_ tokens as query parameters in the callback URL.
3. The application extracts the _identity_ and _access_ tokens from the callback URL.

**Resource Owner Password Credentials Grant (Direct Access Grants)**

This is referred to in the Admin Console as _Direct Access Grants_. This is used by REST clients that want to obtain a token on behalf of a user. It is one HTTP POST request that contains the credentials of the user as well as the id of the client and the client’s secret (if it is a confidential client). The user’s credentials are sent within form parameters. The HTTP response contains _identity_, _access_, and _refresh_ tokens.

**Client Credentials Grant**

This is also used by REST clients, but instead of obtaining a token that works on behalf of an external user, a token is created based on the metadata and permissions of a service account that is associated with the client. More info together with example is in [Service Accounts](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/clients/oidc/service-accounts.html#\_service\_accounts) chapter.

**Keycloak Server OIDC URI Endpoints**

Here’s a list of OIDC endpoints that the Keycloak publishes. These URLs are useful if you are using a non-Keycloak client adapter to talk OIDC with the auth server. These are all relative URLs and the root of the URL being the HTTP(S) protocol, hostname, and usually path prefixed with _/auth_: i.e. https://localhost:8080/auth

/realms/{realm-name}/protocol/openid-connect/token

This is the URL endpoint for obtaining a temporary code in the Authorization Code Flow or for obtaining tokens via the Implicit Flow, Direct Grants, or Client Grants.

/realms/{realm-name}/protocol/openid-connect/auth

This is the URL endpoint for the Authorization Code Flow to turn a temporary code into a token.

/realms/{realm-name}/protocol/openid-connect/logout

This is the URL endpoint for performing logouts.

/realms/{realm-name}/protocol/openid-connect/userinfo

This is the URL endpoint for the User Info service described in the OIDC specification.

In all of these replace _{realm-name}_ with the name of the realm.
