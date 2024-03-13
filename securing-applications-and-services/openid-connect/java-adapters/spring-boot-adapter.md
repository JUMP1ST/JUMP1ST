# Spring Boot Adapter

To be able to secure Spring Boot apps you must add the Keycloak Spring Boot adapter JAR to your app. You then have to provide some extra configuration via normal Spring Boot configuration (`application.properties`). Let’s go over these steps.

**Adapter Installation**

The Keycloak Spring Boot adapter takes advantage of Spring Boot’s autoconfiguration so all you need to do is add the Keycloak Spring Boot starter to your project. They Keycloak Spring Boot Starter is also directly available from the [Spring Start Page](http://start.spring.io/). To add it manually and if you are using Maven, add the following to your dependencies :

```xml
<dependency>
    <groupId>org.keycloak</groupId>
    <artifactId>keycloak-spring-boot-starter</artifactId>
</dependency>
```

Make also sure to add the Adapter BOM dependency :

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.keycloak.bom</groupId>
      <artifactId>keycloak-adapter-bom</artifactId>
      <version>SNAPSHOT</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

Currently the following embedded containers are supported and do not require any extra dependencies if using the Starter:

* Tomcat
* Undertow
* Jetty

**Required Spring Boot Adapter Configuration**

This section describes how to configure your Spring Boot app to use Keycloak.

Instead of a `keycloak.json` file, you configure the realm for the Spring Boot Keycloak adapter via the normal Spring Boot configuration. For example:

```
keycloak.realm = demorealm
keycloak.auth-server-url = http://127.0.0.1:8080/auth
keycloak.ssl-required = external
keycloak.resource = demoapp
keycloak.credentials.secret = 11111111-1111-1111-1111-111111111111
keycloak.use-resource-role-mappings = true
```

You can disable the Keycloak Spring Boot Adapter (for example in tests) by setting `keycloak.enabled = false`.

To configure a Policy Enforcer, unlike keycloak.json, `policy-enforcer-config` must be used instead of just `policy-enforcer`.

You also need to specify the Java EE security config that would normally go in the `web.xml`. The Spring Boot Adapter will set the `login-method` to `KEYCLOAK` and configure the `security-constraints` at startup time. Here’s an example configuration:

```
keycloak.securityConstraints[0].authRoles[0] = admin
keycloak.securityConstraints[0].authRoles[1] = user
keycloak.securityConstraints[0].securityCollections[0].name = insecure stuff
keycloak.securityConstraints[0].securityCollections[0].patterns[0] = /insecure

keycloak.securityConstraints[1].authRoles[0] = admin
keycloak.securityConstraints[1].securityCollections[0].name = admin stuff
keycloak.securityConstraints[1].securityCollections[0].patterns[0] = /admin
```
