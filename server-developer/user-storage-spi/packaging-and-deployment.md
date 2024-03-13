# Packaging and Deployment

User Storage providers are packaged in a JAR and deployed or undeployed to the Keycloak runtime in the same way you would deploy something in the JBoss/Wildfly application server. You can either copy the JAR directly to the `deploy/` directory of the server, or use the JBoss CLI to execute the deployment.

In order for Keycloak to recognize the provider, you need to add a file to the JAR: `META-INF/services/org.keycloak.storage.UserStorageProviderFactory`. This file must contain a line-separated list of fully qualified classnames of the `UserStorageProviderFactory` implementations:

```
org.keycloak.examples.federation.properties.ClasspathPropertiesStorageFactory
org.keycloak.examples.federation.properties.FilePropertiesStorageFactory
```

Keycloak supports hot deployment of these provider JARs. You’ll also see later in this chapter that you can package it within and as Java EE components.
