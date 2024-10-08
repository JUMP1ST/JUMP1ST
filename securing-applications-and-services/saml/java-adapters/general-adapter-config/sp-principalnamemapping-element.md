# SP PrincipalNameMapping element

This element is optional. When creating a Java Principal object that you obtain from methods such as `HttpServletRequest.getUserPrincipal()`, you can define what name is returned by the `Principal.getName()` method.

```xml
<SP ...>
  <PrincipalNameMapping policy="FROM_NAME_ID"/>
</SP>

<SP ...>
  <PrincipalNameMapping policy="FROM_ATTRIBUTE" attribute="email" />
</SP>
```

The `policy` attribute defines the policy used to populate this value. The possible values for this attribute are:

FROM\_NAME\_ID

This policy just uses whatever the SAML subject value is. This is the default setting

FROM\_ATTRIBUTE

This will pull the value from one of the attributes declared in the SAML assertion received from the server. You’ll need to specify the name of the SAML assertion attribute to use within the `attribute` XML attribute.
