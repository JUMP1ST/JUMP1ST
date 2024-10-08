# SAML Clients

Keycloak supports [SAML 2.0](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/sso-protocols/saml.html#\_saml) for registered applications. Both POST and Redirect bindings are supported. You can choose to require client signature validation and can have the server sign and/or encrypt responses as well.

To create a SAML client go to the `Clients` left menu item. On this page you’ll see a `Create` button on the right.

Clients

![clients.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/clients.png)

This will bring you to the `Add Client` page.

Add Client

![add-client-saml.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/add-client-saml.png)

Enter in the `Client ID` of the client. This is often a URL and will be the expected `issuer` value in SAML requests sent by the application. Next select `saml` in the `Client Protocol` drop down box. Ignore the `Client Template` listbox for now, we’ll go over that later in this chapter. Finally enter in the `Client SAML Endpoint` URL. Enter the URL you want the Keycloak server to send SAML requests and responses to. Usually applications have only one URL for processing SAML requests. If your application has different URLs for its bindings, don’t worry, you can fix this in the `Settings` tab of the client. Click `Save`. This will create the client and bring you to the client `Settings` tab.

Client Settings

![client-settings-saml.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/client-settings-saml.png)

Client ID

This value must match the issuer value sent with AuthNRequests. Keycloak will pull the issuer from the Authn SAML request and match it to a client by this value.

Name

This is the display name for the client whenever it is displayed in a Keycloak UI screen. You can localize the value of this field by setting up a replacement string value i.e. ${myapp}. See the [Server Development](https://keycloak.gitbooks.io/documentation/content/server\_development/index.html) for more information.

Description

This specifies the description of the client. This can also be localized.

Enabled

If this is turned off, the client will not be allowed to request authentication.

Consent Required

If this is on, then users will get a consent page which asks the user if they grant access to that application. It will also display the metadata that the client is interested in so that the user knows exactly what information the client is getting access to. If you’ve ever done a social login to Google, you’ll often see a similar page. Keycloak provides the same functionality.

Include AuthnStatement

SAML login responses may specify the authentication method used (password, etc.) as well as a timestamp of the login. Setting this to on will include that statement in the response document.

Sign Documents

When turned on, Keycloak will sign the document using the realm’s private key.

Optimize REDIRECT signing key lookup

When turned on, the SAML protocol messages will include Keycloak native extension that contains a hint with signing key ID. When the SP understands this extension, it can use it for signature validation instead of attempting to validate signature with all known keys. This option only applies to REDIRECT bindings where the signature is transferred in query parameters where there is no place with this information in the signature information (contrary to POST binding messages where key ID is always included in document signature). Currently this is relevant to situations where both IDP and SP are provided by Keycloak server and adapter. This option is only relevant when `Sign Documents` is switched on.

Sign Assertions

The `Sign Documents` switch signs the whole document. With this setting the assertion is also signed and embedded within the SAML XML Auth response.

Signature Algorithm

Choose between a variety of algorithms for signing SAML documents.

SAML Signature Key Name

Signed SAML documents sent via POST binding contain identification of signing key in `KeyName` element. This by default contains Keycloak key ID. However various vendors might expect a different key name or no key name at all. This switch controls whether `KeyName` contains key ID (option `KEY_ID`), subject from certificate corresponding to the realm key (option `CERT_SUBJECT` - expected for instance by Microsoft Active Directory Federation Services), or that the key name hint is completely omitted from the SAML message (option `NONE`).

Canonicalization Method

Canonicalization method for XML signatures.

Encrypt Assertions

Encrypt assertions in SAML documents with the realm’s private key. The AES algorithm is used with a key size of 128 bits.

Client Signature Required

Expect that documents coming from a client are signed. Keycloak will validate this signature using the client public key or cert set up in the `SAML Keys` tab.

Force POST Binding

By default, Keycloak will respond using the initial SAML binding of the original request. By turning on this switch, you will force Keycloak to always respond using the SAML POST Binding even if the original request was the Redirect binding.

Front Channel Logout

If true, this application requires a browser redirect to be able to perform a logout. For example, the application may require a cookie to be reset which could only be done via a redirect. If this switch is false, then Keycloak will invoke a background SAML request to logout the application.

Force Name ID Format

If the request has a name ID policy, ignore it and used the value configured in the admin console under Name ID Format

Name ID Format

Name ID Format for the subject. If no name ID policy is specified in the request or if the Force Name ID Format attribute is true, this value is used. Properties used for each of the respective formats are defined below.

Root URL

If Keycloak uses any configured relative URLs, this value is prepended to them.

Valid Redirect URIs

This is an optional field. Enter in a URL pattern and click the + sign to add. Click the - sign next to URLs you want to remove. Remember that you still have to click the `Save` button! Wildcards (\\\*) are only allowed at the end of of a URI, i.e. http://host.com/\*. This field is used when the exact SAML endpoints are not registered and Keycloak is pull the Assertion Consumer URL from the request.

Base URL

If Keycloak needs to link to the client, this URL would be used.

Master SAML Processing URL

This URL will be used for all SAML requests and the response will be directed to the SP. It will be used as the Assertion Consumer Service URL and the Single Logout Service URL. If a login request contains the Assertion Consumer Service URL, that will take precedence, but this URL must be valided by a registered Valid Redirect URI pattern

Assertion Consumer Service POST Binding URL

POST Binding URL for the Assertion Consumer Service.

Assertion Consumer Service Redirect Binding URL

Redirect Binding URL for the Assertion Consumer Service.

Logout Service POST Binding URL

POST Binding URL for the Logout Service.

Logout Service Redirect Binding URL

Redirect Binding URL for the Logout Service.

\
