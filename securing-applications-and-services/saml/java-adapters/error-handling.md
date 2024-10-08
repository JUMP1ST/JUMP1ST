# Error Handling

Keycloak has some error handling facilities for servlet based client adapters. When an error is encountered in authentication, the client adapter will call `HttpServletResponse.sendError()`. You can set up an `error-page` within your `web.xml` file to handle the error however you want. The client adapter can throw 400, 401, 403, and 500 errors.

```xml
<error-page>
    <error-code>403</error-code>
    <location>/ErrorHandler</location>
</error-page>
```

The client adapter also sets an `HttpServletRequest` attribute that you can retrieve. The attribute name is `org.keycloak.adapters.spi.AuthenticationError`. Typecast this object to: `org.keycloak.adapters.saml.SamlAuthenticationError`. This class can tell you exactly what happened. If this attribute is not set, then the adapter was not responsible for the error code.

```javascript
public class SamlAuthenticationError implements AuthenticationError {
    public static enum Reason {
        EXTRACTION_FAILURE,
        INVALID_SIGNATURE,
        ERROR_STATUS
    }

    public Reason getReason() {
        return reason;
    }
    public StatusResponseType getStatus() {
        return status;
    }
}
```
