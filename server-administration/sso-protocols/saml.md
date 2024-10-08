# SAML

[SAML 2.0](http://saml.xml.org/saml-specifications) is a similar specification to OIDC but a lot older and more mature. It has its roots in SOAP and the plethora of WS-\* specifications so it tends to be a bit more verbose than OIDC. SAML 2.0 is primarily an authentication protocol that works by exchanging XML documents between the authentication server and the application. XML signatures and encryption is used to verify requests and responses.

There is really two types of use cases when using SAML. The first is an application that asks the Keycloak server to authenticate a user for them. After a successful login, the application will receive an XML document that contains something called a SAML assertion that specify various attributes about the user. This XML document is digitally signed by the realm and contains access information (like user role mappings) that the application can use to determine what resources the user is allowed to access on the application.

The second type of use cases is that of a client that wants to gain access to remote services. In this case, the client asks Keycloak to obtain an SAML assertion it can use to invoke on other remote services on behalf of the user.

**SAML Bindings**

SAML defines a few different ways to exchange XML documents when executing the authentication protocol. The _Redirect_ and _Post_ bindings cover browser based applications. The _ECP_ binding covers REST invocations. There are other binding types but Keycloak only supports those three.

**Redirect Binding**

The _Redirect_ Binding uses a series of browser redirect URIs to exchange information. This is a rough overview of how it works.

1. The user visits the application and the application finds the user is not authenticated. It generates an XML authentication request document and encodes it as a query param in a URI that is used to redirect to the Keycloak server. Depending on your settings, the application may also digitally sign this XML document and also stuff this signature as a query param in the redirect URI to Keycloak. This signature is used to validate the client that sent this request.
2. The browser is redirected to Keycloak. The server extracts the XML auth request document and verifies the digital signature if required. The user then has to enter in their credentials to be authenticated.
3. After authentication, the server generates an XML authentication response document. This document contains a SAML assertion that holds metadata about the user like name, address, email, and any role mappings the user might have. This document is almost always digitally signed using XML signatures, and may also be encrypted.
4. The XML auth response document is then encoded as a query param in a redirect URI that brings the browser back to the application. The digital signature is also included as a query param.
5. The application receives the redirect URI and extracts the XML document and verifies the realm’s signature to make sure it is receiving a valid auth response. The information inside the SAML assertion is then used to make access decisions or display user data.

**POST Binding**

The SAML _POST_ binding works almost the exact same way as the _Redirect_ binding, but instead of GET requests, XML documents are exchanged by POST requests. The _POST_ Binding uses JavaScript to trick the browser into making a POST request to the Keycloak server or application when exchanging documents. Basically HTTP responses contain an HTML document that contains an HTML form with embedded JavaScript. When the page is loaded, the JavaScript automatically invokes the form. You really don’t need to know about this stuff, but it is a pretty clever trick.

**ECP**

ECP stands for "Enhanced Client or Proxy", a SAML v.2.0 profile which allows for the exchange of SAML attributes outside the context of a web browser. This is used most often for REST or SOAP-based clients.

**Keycloak Server SAML URI Endpoints**

Keycloak really only has one endpoint for all SAML requests.

`http(s)://authserver.host/auth/realms/{realm-name}/protocol/saml`

All bindings use this endpoint.
