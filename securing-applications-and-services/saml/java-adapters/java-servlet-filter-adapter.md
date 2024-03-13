# Java Servlet Filter Adapter

If you want to use SAML with a Java servlet application that doesn’t have an adapter for that servlet platform, you can opt to use the servlet filter adapter that Keycloak has. This adapter works a little differently than the other adapters. You still have to specify a `/WEB-INF/keycloak-saml.xml` file as defined in the [General Adapter Config](https://wjw465150.gitbooks.io/keycloak-documentation/content/securing\_apps/saml/java/general-config.html#%3Cem%3Esaml-general-config) section, but you do not define security constraints in \_web.xml. Instead you define a filter mapping using the Keycloak servlet filter adapter to secure the url patterns you want to secure.

| Note | Backchannel logout works a bit differently than the standard adapters. Instead of invalidating the http session it instead marks the session ID as logged out. There’s just no way of arbitrarily invalidating an http session based on a session ID. |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

| Warning | Backchannel logout does not currently work when you have a clustered application that uses the SAML filter. |
| ------- | ----------------------------------------------------------------------------------------------------------- |

```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
      version="3.0">

	<module-name>customer-portal</module-name>

    <filter>
        <filter-name>Keycloak Filter</filter-name>
        <filter-class>org.keycloak.adapters.saml.servlet.SamlFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>Keycloak Filter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```

The Keycloak filter has the same configuration parameters available as the other adapters except you must define them as filter init params instead of context params.

You can define multiple filter mappings if you have various different secure and unsecure url patterns.

| Warning | You must have a filter mapping that covers `/saml`. This mapping covers all server callbacks. |
| ------- | --------------------------------------------------------------------------------------------- |

When registering SPs with an IdP, you must register `http[s]://hostname/{context-root}/saml` as your Assert Consumer Service URL and Single Logout Service URL.

To use this filter, include this maven artifact in your WAR poms:

```xml
<dependency>
   <groupId>org.keycloak</groupId>
   <artifactId>keycloak-saml-servlet-filter-adapter</artifactId>
   <version>SNAPSHOT</version>
</dependency>
```

\
