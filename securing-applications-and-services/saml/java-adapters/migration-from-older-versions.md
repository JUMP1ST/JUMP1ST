# Migration from older versions

**Migrating to 1.9.0**

**SAML SP Client Adapter Changes**

Keycloak SAML SP Client Adapter now requires a specific endpoint, `/saml` to be registered with your IdP. The SamlFilter must also be bound to /saml in addition to any other binding it has. This had to be done because SAML POST binding would eat the request input stream and this would be really bad for clients that relied on it.
