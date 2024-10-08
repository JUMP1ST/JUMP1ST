# RoleIdentifiers Element

The `RoleIdentifiers` element defines what SAML attributes within the assertion received from the user should be used as role identifiers within the Java EE Security Context for the user.

```xml
<RoleIdentifiers>
     <Attribute name="Role"/>
     <Attribute name="member"/>
     <Attribute name="memberOf"/>
</RoleIdentifiers>
```

By default `Role` attribute values are converted to Java EE roles. Some IdPs send roles using a `member` or `memberOf` attribute assertion. You can define one or more `Attribute` elements to specify which SAML attributes must be converted into roles.
