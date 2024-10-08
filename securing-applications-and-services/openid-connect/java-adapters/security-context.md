# Security Context

The `KeycloakSecurityContext` interface is available if you need to access to the tokens directly. This could be useful if you want to retrieve additional details from the token (such as user profile information) or you want to invoke a RESTful service that is protected by Keycloak.

In servlet environments it is available in secured invocations as an attribute in HttpServletRequest:

```java
httpServletRequest
    .getAttribute(KeycloakSecurityContext.class.getName());
```

Or, it is available in secure and insecure requests in the HttpSession:

```java
httpServletRequest.getSession()
    .getAttribute(KeycloakSecurityContext.class.getName());
```
