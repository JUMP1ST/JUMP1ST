# IDP SingleLogoutService sub element

The `SingleLogoutService` sub element defines the logout SAML endpoint of the IDP. The client adapter will send requests to the IDP formatted via the settings within this element when it wants to logout.

```xml
<SingleLogoutService validateRequestSignature="true"
                     validateResponseSignature="true"
                     signRequest="true"
                     signResponse="true"
                     requestBinding="redirect"
                     responseBinding="post"
                     postBindingUrl="posturl"
                     redirectBindingUrl="redirecturl">
```

signRequest

Should the client sign logout requests it makes to the IDP? This setting is _OPTIONAL_. Defaults to whatever the IDP `signaturesRequired` element value is.

signResponse

Should the client sign logout responses it sends to the IDP requests? This setting is _OPTIONAL_. Defaults to whatever the IDP `signaturesRequired` element value is.

validateRequestSignature

Should the client expect signed logout request documents from the IDP? This setting is _OPTIONAL_. Defaults to whatever the IDP `signaturesRequired` element value is.

validateResponseSignature

Should the client expect signed logout response documents from the IDP? This setting is _OPTIONAL_. Defaults to whatever the IDP `signaturesRequired` element value is.

requestBinding

This is the SAML binding type used for communicating SAML requests to the IDP. This setting is _OPTIONAL_. The default value is `POST`, but you can set it to REDIRECT as well.

responseBinding

This is the SAML binding type used for communicating SAML responses to the IDP. The values of this can be `POST` or `REDIRECT`. This setting is _OPTIONAL_. The default value is `POST`, but you can set it to `REDIRECT` as well.

postBindingUrl

This is the URL for the IDP’s logout service when using the POST binding. This setting is _REQUIRED_ if using the `POST` binding.

redirectBindingUrl

This is the URL for the IDP’s logout service when using the REDIRECT binding. This setting is _REQUIRED_ if using the REDIRECT binding.
