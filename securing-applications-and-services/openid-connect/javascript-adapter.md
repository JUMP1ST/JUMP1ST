# Javascript Adapter

Keycloak comes with a client-side JavaScript library that can be used to secure HTML5/JavaScript applications. The JavaScript adapter has built-in support for Cordova applications.

The library can be retrieved directly from the Keycloak server at `/auth/js/keycloak.js` and is also distributed as a ZIP archive.

A best practice is to load the JavaScript adapter directly from Keycloak Server as it will automatically be updated when you upgrade the server. If you copy the adapter to your web application instead, make sure you upgrade the adapter only after you have upgraded the server.

One important thing to note about using client-side applications is that the client has to be a public client as there is no secure way to store client credentials in a client-side application. This makes it very important to make sure the redirect URIs you have configured for the client are correct and as specific as possible.

To use the JavaScript adapter you must first create a client for your application in the Keycloak Administration Console. Make sure `public` is selected for `Access Type`.

You also need to configure valid redirect URIs and valid web origins. Be as specific as possible as failing to do so may result in a security vulnerability.

Once the client is created click on the `Installation` tab select `Keycloak OIDC JSON` for `Format Option` then click `Download`. The downloaded `keycloak.json` file should be hosted on your web server at the same location as your HTML pages.

Alternatively, you can skip the configuration file and manually configure the adapter.

The following example shows how to initialize the JavaScript adapter:

```html
<head>
    <script src="keycloak.js"></script>
    <script>
        var keycloak = Keycloak();
        keycloak.init().success(function(authenticated) {
            alert(authenticated ? 'authenticated' : 'not authenticated');
        }).error(function() {
            alert('failed to initialize');
        });
    </script>
</head>
```

If the `keycloak.json` file is in a different location you can specify it:

```javascript
var keycloak = Keycloak('http://localhost:8080/myapp/keycloak.json'));
```

Alternatively, you can pass in a JavaScript object with the required configuration instead:

```javascript
var keycloak = Keycloak({
    url: 'http://keycloak-server/auth',
    realm: 'myrealm',
    clientId: 'myapp'
});
```

By default to authenticate you need to call the `login` function. However, there are two options available to make the adapter automatically authenticate. You can pass `login-required` or `check-sso` to the init function. `login-required` will authenticate the client if the user is logged-in to Keycloak or display the login page if not. `check-sso` will only authenticate the client if the user is already logged-in, if the user is not logged-in the browser will be redirected back to the application and remain unauthenticated.

To enable `login-required` set `onLoad` to `login-required` and pass to the init method:

```
keycloak.init({ onLoad: 'login-required' })
```

After the user is authenticated the application can make requests to RESTful services secured by Keycloak by including the bearer token in the `Authorization` header. For example:

```javascript
var loadData = function () {
    document.getElementById('username').innerText = keycloak.subject;

    var url = 'http://localhost:8080/restful-service';

    var req = new XMLHttpRequest();
    req.open('GET', url, true);
    req.setRequestHeader('Accept', 'application/json');
    req.setRequestHeader('Authorization', 'Bearer ' + keycloak.token);

    req.onreadystatechange = function () {
        if (req.readyState == 4) {
            if (req.status == 200) {
                alert('Success');
            } else if (req.status == 403) {
                alert('Forbidden');
            }
        }
    }

    req.send();
};
```

One thing to keep in mind is that the access token by default has a short life expiration so you may need to refresh the access token prior to sending the request. You can do this by the `updateToken` method. The `updateToken` method returns a promise object which makes it easy to invoke the service only if the token was successfully refreshed and for example display an error to the user if it wasn’t. For example:

```javascript
keycloak.updateToken(30).success(function() {
    loadData();
}).error(function() {
    alert('Failed to refresh token');
);
```

**Session Status iframe**

By default, the JavaScript adapter creates a hidden iframe that is used to detect if a Single-Sign Out has occurred. This does not require any network traffic, instead the status is retrieved by looking at a special status cookie. This feature can be disabled by setting `checkLoginIframe: false` in the options passed to the `init` method.

You should not rely on looking at this cookie directly. It’s format can change and it’s also associated with the URL of the Keycloak server, not your application.

**Implicit and Hybrid Flow**

By default, the JavaScript adapter uses the [Authorization Code](http://openid.net/specs/openid-connect-core-1\_0.html#CodeFlowAuth) flow.

With this flow the Keycloak server returns an authorization code, not an authentication token, to the application. The JavaScript adapter exchanges the `code` for an access token and a refresh token after the browser is redirected back to the application.

Keycloak also supports the [Implicit](http://openid.net/specs/openid-connect-core-1\_0.html#ImplicitFlowAuth) flow where an access token is sent immediately after successful authentication with Keycloak. This may have better performance than standard flow, as there is no additional request to exchange the code for tokens, but it has implications when the access token expires.

However, sending the access token in the URL fragment can be a security vulnerability. For example the token could be leaked through web server logs and or browser history.

To enable implicit flow, you need to enable the `Implicit Flow Enabled` flag for the client in the Keycloak Administration Console. You also need to pass the parameter `flow` with value `implicit` to `init` method:

```javascript
keycloak.init({ flow: 'implicit' })
```

One thing to note is that only an access token is provided and there is no refresh token. This means that once the access token has expired the application has to do the redirect to the Keycloak again to obtain a new access token.

Keycloak also supports the [Hybrid](http://openid.net/specs/openid-connect-core-1\_0.html#HybridFlowAuth) flow.

This requires the client to have both the `Standard Flow Enabled` and `Implicit Flow Enabled` flags enabled in the admin console. The Keycloak server will then send both the code and tokens to your application. The access token can be used immediately while the code can be exchanged for access and refresh tokens. Similar to the implicit flow, the hybrid flow is good for performance because the access token is available immediately. But, the token is still sent in the URL, and the security vulnerability mentioned earlier may still apply.

One advantage in the Hybrid flow is that the refresh token is made available to the application.

For the Hybrid flow, you need to pass the parameter `flow` with value `hybrid` to the `init` method:

```javascript
keycloak.init({ flow: 'hybrid' })
```

**Earlier Browsers**

The JavaScript adapter depends on Base64 (window.btoa and window.atob) and HTML5 History API. If you need to support browsers that do not have these available (for example, IE9) you need to add polyfillers.

Example polyfill libraries:

* [https://github.com/davidchambers/Base64.js](https://github.com/davidchambers/Base64.js)
* [https://github.com/devote/HTML5-History-API](https://github.com/devote/HTML5-History-API)

**JavaScript Adapter Reference**

**Constructor**

```javascript
new Keycloak();
new Keycloak('http://localhost/keycloak.json');
new Keycloak({ url: 'http://localhost/auth', realm: 'myrealm', clientId: 'myApp' });
```

**Properties**

authenticated

Is `true` if the user is authenticated, `false` otherwise.

token

The base64 encoded token that can be sent in the `Authorization` header in requests to services.

tokenParsed

The parsed token as a JavaScript object.

subject

The user id.

idToken

The base64 encoded ID token.

idTokenParsed

The parsed id token as a JavaScript object.

realmAccess

The realm roles associated with the token.

resourceAccess

The resource roles associated with the token.

refreshToken

The base64 encoded refresh token that can be used to retrieve a new token.

refreshTokenParsed

The parsed refresh token as a JavaScript object.

timeSkew

The estimated time difference between the browser time and the Keycloak server in seconds. This value is just an estimation, but is accurate enough when determining if a token is expired or not.

responseMode

Response mode passed in init (default value is fragment).

flow

Flow passed in init.

responseType

Response type sent to Keycloak with login requests. This is determined based on the flow value used during initialization, but can be overridden by setting this value.

**Methods**

**init(options)**

Called to initialize the adapter.

Options is an Object, where:

* onLoad - Specifies an action to do on load. Supported values are 'login-required' or 'check-sso'.
* token - Set an initial value for the token.
* refreshToken - Set an initial value for the refresh token.
* idToken - Set an initial value for the id token (only together with token or refreshToken).
* timeSkew - Set an initial value for skew between local time and Keycloak server in seconds (only together with token or refreshToken).
* checkLoginIframe - Set to enable/disable monitoring login state (default is true).
* checkLoginIframeInterval - Set the interval to check login state (default is 5 seconds).
* responseMode - Set the OpenID Connect response mode send to Keycloak server at login request. Valid values are query or fragment . Default value is fragment, which means that after successful authentication will Keycloak redirect to javascript application with OpenID Connect parameters added in URL fragment. This is generally safer and recommended over query.
* flow - Set the OpenID Connect flow. Valid values are standard, implicit or hybrid.

Returns promise to set functions to be invoked on success or error.

**login(options)**

Redirects to login form on (options is an optional object with redirectUri and/or prompt fields).

Options is an Object, where:

* redirectUri - Specifies the uri to redirect to after login.
* prompt - By default the login screen is displayed if the user is not logged-in to Keycloak. To only authenticate to the application if the user is already logged-in and not display the login page if the user is not logged-in, set this option to `none`. To always require re-authentication and ignore SSO, set this option to `login` .
* maxAge - Used just if user is already authenticated. Specifies maximum time since the authentication of user happened. If user is already authenticated for longer time than `maxAge`, the SSO is ignored and he will need to re-authenticate again.
* loginHint - Used to pre-fill the username/email field on the login form.
* action - If value is 'register' then user is redirected to registration page, otherwise to login page.
* locale - Specifies the desired locale for the UI.

**createLoginUrl(options)**

Returns the URL to login form on (options is an optional object with redirectUri and/or prompt fields).

Options is an Object, which supports same options like the function `login` .

**logout(options)**

Redirects to logout.

Options is an Object, where:

* redirectUri - Specifies the uri to redirect to after logout.

**createLogoutUrl(options)**

Returns the URL to logout the user.

Options is an Object, where:

* redirectUri - Specifies the uri to redirect to after logout.

**register(options)**

Redirects to registration form. Shortcut for login with option action = 'register'

Options are same as for the login method but 'action' is set to 'register'

**createRegisterUrl(options)**

Returns the url to registration page. Shortcut for createLoginUrl with option action = 'register'

Options are same as for the createLoginUrl method but 'action' is set to 'register'

**accountManagement()**

Redirects to the Account Management Console.

**createAccountUrl()**

Returns the URL to the Account Management Console.

**hasRealmRole(role)**

Returns true if the token has the given realm role.

**hasResourceRole(role, resource)**

Returns true if the token has the given role for the resource (resource is optional, if not specified clientId is used).

**loadUserProfile()**

Loads the users profile.

Returns promise to set functions to be invoked if the profile was loaded successfully, or if the profile could not be loaded.

For example:

```javascript
keycloak.loadUserProfile().success(function(profile) {
        alert(JSON.stringify(test, null, "  "));
    }).error(function() {
        alert('Failed to load user profile');
    });
```

**isTokenExpired(minValidity)**

Returns true if the token has less than minValidity seconds left before it expires (minValidity is optional, if not specified 0 is used).

**updateToken(minValidity)**

If the token expires within minValidity seconds (minValidity is optional, if not specified 5 is used) the token is refreshed. If the session status iframe is enabled, the session status is also checked.

Returns promise to set functions that can be invoked if the token is still valid, or if the token is no longer valid. For example:

```javascript
keycloak.updateToken(5).success(function(refreshed) {
        if (refreshed) {
            alert('Token was successfully refreshed');
        } else {
            alert('Token is still valid');
        }
    }).error(function() {
        alert('Failed to refresh the token, or the session has expired');
    });
```

**clearToken()**

Clear authentication state, including tokens. This can be useful if application has detected the session was expired, for example if updating token fails.

Invoking this results in onAuthLogout callback listener being invoked.

**Callback Events**

The adapter supports setting callback listeners for certain events.

For example:

```javascript
keycloak.onAuthSuccess = function() { alert('authenticated'); }
```

The available events are:

* onReady(authenticated) - Called when the adapter is initialized.
* onAuthSuccess - Called when a user is successfully authenticated.
* onAuthError - Called if there was an error during authentication.
* onAuthRefreshSuccess - Called when the token is refreshed.
* onAuthRefreshError - Called if there was an error while trying to refresh the token.
* onAuthLogout - Called if the user is logged out (will only be called if the session status iframe is enabled, or in Cordova mode).
* onTokenExpired - Called when the access token is expired. If a refresh token is available the token can be refreshed with updateToken, or in cases where it is not (that is, with implicit flow) you can redirect to login screen to obtain a new access token.

\
