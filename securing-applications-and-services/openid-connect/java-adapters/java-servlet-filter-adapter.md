# Java Servlet Filter Adapter

If you are deploying your Java Servlet application on a platform where there is no Keycloak adapter you opt to use the servlet filter adapter. This adapter works a bit differently than the other adapters. You do not define security constraints in web.xml. Instead you define a filter mapping using the Keycloak servlet filter adapter to secure the url patterns you want to secure.

| Warning | Backchannel logout works a bit differently than the standard adapters. Instead of invalidating the HTTP session it marks the session id as logged out. There’s no standard way to invalidate an HTTP session based on a session id. |
| ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
      version="3.0">

	<module-name>application</module-name>

    <filter>
        <filter-name>Keycloak Filter</filter-name>
        <filter-class>org.keycloak.adapters.servlet.KeycloakOIDCFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>Keycloak Filter</filter-name>
        <url-pattern>/keycloak/*</url-pattern>
        <url-pattern>/protected/*</url-pattern>
    </filter-mapping>
</web-app>
```

In the snippet above there are two url-patterns. _/protected/\*_ are the files we want protected, while the _/keycloak/\*_ url-pattern handles callbacks from the Keycloak server.

If you need to exclude some paths beneath the configured `url-patterns` you can use the Filter init-param `keycloak.config.skipPattern` to configure a regular expression that describes a path-pattern for which the keycloak filter should immediately delegate to the filter-chain. By default no skipPattern is configured.

Patterns are matched against the `requestURI` without the `context-path`. Given the context-path `/myapp` a request for `/myapp/index.html` will be matched with `/index.html` against the skip pattern.

```xml
<init-param>
    <param-name>keycloak.config.skipPattern</param-name>
    <param-value>^/(path1|path2|path3).*</param-value>
</init-param>
```

Note that you should configure your client in the Keycloak Admin Console with an Admin URL that points to a secured section covered by the filter’s url-pattern.

The Admin URL will make callbacks to the Admin URL to do things like backchannel logout. So, the Admin URL in this example should be `http[s]://hostname/{context-root}/keycloak`.

The Keycloak filter has the same configuration parameters as the other adapters except you must define them as filter init params instead of context params.

To use this filter, include this maven artifact in your WAR poms:

```xml
<dependency>
    <groupId>org.keycloak</groupId>
    <artifactId>keycloak-servlet-filter-adapter</artifactId>
    <version>SNAPSHOT</version>
</dependency>
```

\
