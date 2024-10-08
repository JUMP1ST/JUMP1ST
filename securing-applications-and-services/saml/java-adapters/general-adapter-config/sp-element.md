# SP Element

Here is the explanation of the SP element attributes:

```xml
<SP entityID="sp"
    sslPolicy="ssl"
    nameIDPolicyFormat="format"
    forceAuthentication="true"
    isPassive="false"
    autodetectBearerOnly="false">
...
</SP>
```

entityID

This is the identifier for this client. The IdP needs this value to determine who the client is that is communicating with it. This setting is _REQUIRED_.

sslPolicy

This is the SSL policy the adapter will enforce. Valid values are: `ALL`, `EXTERNAL`, and `NONE`. For `ALL`, all requests must come in via HTTPS. For `EXTERNAL`, only non-private IP addresses must come over the wire via HTTPS. For `NONE`, no requests are required to come over via HTTPS. This setting is _OPTIONAL_. Default value is `EXTERNAL`.

nameIDPolicyFormat

SAML clients can request a specific NameID Subject format. Fill in this value if you want a specific format. It must be a standard SAML format identifier: `urn:oasis:names:tc:SAML:2.0:nameid-format:transient`. This setting is _OPTIONAL_. By default, no special format is requested.

forceAuthentication

SAML clients can request that a user is re-authenticated even if they are already logged in at the IdP. Set this to `true` to enable. This setting is _OPTIONAL_. Default value is `false`.

isPassive

SAML clients can request that a user is never asked to authenticate even if they are not logged in at the IdP. Set this to `true` if you want this. Do not use together with `forceAuthentication` as they are opposite. This setting is _OPTIONAL_. Default value is `false`.

turnOffChangeSessionIdOnLogin

The session ID is changed by default on a successful login on some platforms to plug a security attack vector. Change this to `true` to disable this. It is recommended you do not turn it off. Default value is `false`.

autodetectBearerOnly

This should be set to _true_ if your application serves both a web application and web services (e.g. SOAP or REST). It allows you to redirect unauthenticated users of the web application to the Keycloak login page, but send an HTTP `401` status code to unauthenticated SOAP or REST clients instead as they would not understand a redirect to the login page. Keycloak auto-detects SOAP or REST clients based on typical headers like `X-Requested-With`, `SOAPAction` or `Accept`. The default value is _false_.
