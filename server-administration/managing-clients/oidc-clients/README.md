# OIDC Clients

[OpenID Connect](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/sso-protocols/oidc.html#\_oidc) is the preferred protocol to secure applications. It was designed from the ground up to be web friendly and work best with HTML5/JavaScript applications.

To create an OIDC client go to the `Clients` left menu item. On this page you’ll see a `Create` button on the right.

Clients

![clients.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/clients.png)

This will bring you to the `Add Client` page.

Add Client

![add-client-oidc.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/add-client-oidc.png)

Enter in the `Client ID` of the client. This should be a simple alpha-numeric string that will be used in requests and in the Keycloak database to identity the client. Next select `openid-connect` in the `Client Protocol` drop down box. Ignore the `Client Template` listbox for now, we’ll go over that later in this chapter. Finally enter in the base URL of your application in the `Root URL` field and click `Save`. This will create the client and bring you to the client `Settings` tab.

Client Settings

![client-settings-oidc.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/client-settings-oidc.png)

Let’s walk through each configuration item on this page.

**Client ID**

This specifies an alpha-numeric string that will be used as the client identifier for OIDC requests.

**Name**

This is the display name for the client whenever it is displayed in a Keycloak UI screen. You can localize the value of this field by setting up a replacement string value i.e. ${myapp}. See the [Server Development](https://keycloak.gitbooks.io/documentation/content/server\_development/index.html) for more information.

**Description**

This specifies the description of the client. This can also be localized.

**Enabled**

If this is turned off, the client will not be allowed to request authentication.

**Consent Required**

If this is on, then users will get a consent page which asks the user if they grant access to that application. It will also display the metadata that the client is interested in so that the user knows exactly what information the client is getting access to. If you’ve ever done a social login to Google, you’ll often see a similar page. Keycloak provides the same functionality.

**Access Type**

This defines the type of the OIDC client.

_confidential_

Confidential access type is for server-side clients that need to perform a browser login and require a client secret when they turn an access code into an access token, (see [Access Token Request](http://tools.ietf.org/html/rfc6749#section-4.1.3) in the OAuth 2.0 spec for more details). This type should be used for server-side applications.

_public_

Public access type is for client-side clients that need to perform a browser login. With a client-side application there is no way to keep a secret safe. Instead it is very important to restrict access by configuring correct redirect URIs for the client.

_bearer-only_

Bearer-only access type means that the application only allows bearer token requests. If this is turned on, this application cannot participate in browser logins.

**Root URL**

If Keycloak uses any configured relative URLs, this value is prepended to them.

**Valid Redirect URIs**

This is a required field. Enter in a URL pattern and click the + sign to add. Click the - sign next to URLs you want to remove. Remember that you still have to click the `Save` button! Wildcards (\\\*) are only allowed at the end of a URI, i.e. http://host.com/\*

You should take extra precautions when registering valid redirect URI patterns. If you make them too general you are vulnerable to attacks. See [Threat Model Mitigation](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/threat/redirect.html#\_unspecific-redirect-uris) chapter for more information.

**Base URL**

If Keycloak needs to link to the client, this URL is used.

**Standard Flow Enabled**

If this is on, clients are allowed to use the OIDC [Authorization Code Flow](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/sso-protocols/oidc.html#\_oidc-auth-flows).

**Implicit Flow Enabled**

If this is on, clients are allowed to use the OIDC [Implicit Flow](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/sso-protocols/oidc.html#\_oidc-auth-flows).

**Direct Grants Enabled**

If this is on, clients are allowed to use the OIDC [Direct Grants](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/sso-protocols/oidc.html#\_oidc-auth-flows).

**Admin URL**

For Keycloak specific client adapters, this is the callback endpoint for the client. The Keycloak server will use this URI to make callbacks like pushing revocation policies, performing backchannel logout, and other administrative operations. For Keycloak servlet adapters, this can be the root URL of the servlet application. For more information see [Securing Applications and Services Guide](https://keycloak.gitbooks.io/documentation/content/securing\_apps/index.html).

**Web Origins**

This option centers around [CORS](http://www.w3.org/TR/cors/) which stands for Cross-Origin Resource Sharing. If browser JavaScript tries to make an AJAX HTTP request to a server whose domain is different from the one the JavaScript code came from, then the request must use CORS. The server must handle CORS requests in a special way, otherwise the browser will not display or allow the request to be processed. This protocol exists to protect against XSS, CSRF and other JavaScript-based attacks.

Keycloak has support for validated CORS requests. The way it works is that the domains listed in the `Web Origins` setting for the client are embedded within the access token sent to the client application. The client application can then use this information to decide whether or not to allow a CORS request to be invoked on it. This is an extension to the OIDC protocol so only Keycloak client adapters support this feature. See [Securing Applications and Services Guide](https://keycloak.gitbooks.io/documentation/content/securing\_apps/index.html) for more information.

To fill in the `Web Origins` data, enter in a base URL and click the + sign to add. Click the - sign next to URLs you want to remove. Remember that you still have to click the `Save` button!
