# JavaScript Integration

The Keycloak Server comes with a JavaScript library you can use to interact with a resource server protected by a policy enforcer. This library is based on the Keycloak JavaScript adapter, which can be integrated to allow your client to obtain permissions from a Keycloak Server.

You can obtain this library from a running a Keycloak Server instance by including the following `script` tag in your web page:

```html
<script src="http://.../auth/js/keycloak-authz.js"></script>
```

Once you do that, you can create a `KeycloakAuthorization` instance as follows:

```javascript
var keycloak = ... // obtain a Keycloak instance from keycloak.js library
var authorization = new KeycloakAuthorization(keycloak);
```

The **keycloak-authz.js** library provides two main features:

* Handle responses from a resource server protected by a [Keycloak Policy Enforcer](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/enforcer/overview.html#\_enforcer\_overview) and obtain a requesting party token (RPT) with the necessary permissions to gain access to the protected resources on the resource server.
  * In this case, the library can handle whatever authorization protocol the resource server is using: [UMA](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/service/authorization/authorization-api.html#\_service\_authorization\_api) or [Entitlements](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/service/entitlement/entitlement-api.html#\_service\_entitlement\_api).
* Obtain permissions from a Keycloak Server using the [Entitlement API](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/service/entitlement/entitlement-api.html#\_service\_entitlement\_api).

In both cases, the library allows you to easily interact with both resource server and Keycloak to obtain tokens with permissions your client can use as bearer tokens to access the protected resources on a resource server.

**Handling Authorization Responses from a Resource Server**

If a resource server is protected by a policy enforcer, it responds to client requests based on the permissions carried along with a [bearer token](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/enforcer/keycloak-enforcement-bearer.html#\_enforcer\_bearer). Typically, when you try to access a resource server with a bearer token that is lacking permissions to access a protected resource, the resource server responds with a **401** status code and a `WWW-Authenticate` header.

The value of the `WWW-Authenticate` header depends on the authorization protocol in use by the resource server. Whatever protocol is in use, you can use a `KeycloakAuthorization` instance to handle responses as follows:

```javascript
var wwwAuthenticateHeader = ... // extract WWW-Authenticate Header from the response in case of a 401 status code
authorization.authorize(wwwAuthenticateHeader).then(function (rpt) {
    // onGrant callback function.
    // If authorization was successful you'll receive an RPT
    // with the necessary permissions to access the resource server
}, function () {
    // onDeny callback function.
    // Called when the authorization request is denied by the server
}, function () {
    // onError callback function. Called when the server responds unexpectedly
});
```

The `authorize` function is completely asynchronous and supports a few callback functions to receive notifications from the server:

* `onGrant`: The first argument of the function. If authorization was successful and the server returned an RPT with the requested permissions, the callback receives the RPT.
* `onDeny`: The second argument of the function. Only called if the server has denied the authorization request.
* `onError`: The third argument of the function. Only called if the server responds unexpectedly.

Most applications should use the `onGrant` callback to retry a request after a 401 response. Subsequent requests should include the RPT as a bearer token for retries.

**Obtaining Entitlements**

The keycloak-authz.js library provides an `entitlement` function that you can use to obtain an RPT from the server using the Entitlement API.

```json
authorization.entitlement('my-resource-server-id').then(function (rpt) {
    // onGrant callback function.
    // If authorization was successful you'll receive an RPT
    // with the necessary permissions to access the resource server
});
```

When using the `entitlement` function, you must provide the _client\_id_ of the resource server you want to access.

The `entitlement` function is completely asynchronous and supports a few callback functions to receive notifications from the server:

* `onGrant`: The first argument of the function. If authorization was successful and the server returned an RPT with the requested permissions, the callback receives the RPT.
* `onDeny`: The second argument of the function. Only called if the server has denied the authorization request.
* `onError`: The third argument of the function. Only called if the server responds unexpectedly.

**Obtaining the RPT**

If you have already obtained an RPT using any of the authorization functions provided by the library, you can always obtain the RPT as follows from the authorization object (assuming that it has been initialized by one of the techniques shown earlier):

```javascript
var rpt = authorization.rpt;
```

\
