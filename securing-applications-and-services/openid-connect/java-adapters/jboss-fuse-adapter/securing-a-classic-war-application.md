# Securing a Classic WAR Application

The needed steps to secure your WAR application are:

1. In the `/WEB-INF/web.xml` file, declare the necessary:
   * security constraints in the \<security-constraint> element
   * login configuration in the \<login-config> element
   *   security roles in the \<security-role> element.

       For example:

       ```xml
       <?xml version="1.0" encoding="UTF-8"?>
       <web-app xmlns="http://java.sun.com/xml/ns/javaee"
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
                version="3.0">

           <module-name>customer-portal</module-name>

           <welcome-file-list>
               <welcome-file>index.html</welcome-file>
           </welcome-file-list>

           <security-constraint>
               <web-resource-collection>
                   <web-resource-name>Customers</web-resource-name>
                   <url-pattern>/customers/*</url-pattern>
               </web-resource-collection>
               <auth-constraint>
                   <role-name>user</role-name>
               </auth-constraint>
           </security-constraint>

           <login-config>
               <auth-method>BASIC</auth-method>
               <realm-name>does-not-matter</realm-name>
           </login-config>

           <security-role>
               <role-name>admin</role-name>
           </security-role>
           <security-role>
               <role-name>user</role-name>
           </security-role>
       </web-app>
       ```
2.  Add the `jetty-web.xml` file with the authenticator to the `/WEB-INF/jetty-web.xml` file.

    For example:

    ```xml
    <?xml version="1.0"?>
    <!DOCTYPE Configure PUBLIC "-//Mort Bay Consulting//DTD Configure//EN"
     "http://www.eclipse.org/jetty/configure_9_0.dtd">
    <Configure class="org.eclipse.jetty.webapp.WebAppContext">
        <Get name="securityHandler">
            <Set name="authenticator">
                <New class="org.keycloak.adapters.jetty.KeycloakJettyAuthenticator">
                </New>
            </Set>
        </Get>
    </Configure>
    ```
3. Within the `/WEB-INF/` directory of your WAR, create a new file, keycloak.json. The format of this configuration file is described in the [Java Adapters Config](https://wjw465150.gitbooks.io/keycloak-documentation/content/securing\_apps/topics/oidc/java/java-adapter-config.html#\_java\_adapter\_config) section. It is also possible to make this file available externally as described in [Configuring the External Adapter](https://wjw465150.gitbooks.io/keycloak-documentation/content/securing\_apps/topics/oidc/java/fuse/classic-war.html#config\_external\_adapter).
4.  Ensure your WAR application imports `org.keycloak.adapters.jetty` and maybe some more packages in the `META-INF/MANIFEST.MF` file, under the `Import-Package` header. Using `maven-bundle-plugin` in your project properly generates OSGI headers in manifest. Note that "\*" resolution for the package does not import the `org.keycloak.adapters.jetty` package, since it is not used by the application or the Blueprint or Spring descriptor, but is rather used in the `jetty-web.xml` file.

    The list of the packages to import might look like this:

    ```
    org.keycloak.adapters.jetty;version="SNAPSHOT",
    org.keycloak.adapters;version="SNAPSHOT",
    org.keycloak.constants;version="SNAPSHOT",
    org.keycloak.util;version="SNAPSHOT",
    org.keycloak.*;version="SNAPSHOT",
    *;resolution:=optional
    ```

**Configuring the External Adapter**

If you do not want the `keycloak.json` adapter configuration file to be bundled inside your WAR application, but instead made available externally and loaded based on naming conventions, use this configuration method.

To enable the functionality, add this section to your `/WEB_INF/web.xml` file:

```xml
<context-param>
    <param-name>keycloak.config.resolver</param-name>
    <param-value>org.keycloak.adapters.osgi.PathBasedKeycloakConfigResolver</param-value>
</context-param>
```

That component uses `keycloak.config` or `karaf.etc` java properties to search for a base folder to locate the configuration. Then inside one of those folders it searches for a file called `<your_web_context>-keycloak.json`.

So, for example, if your web application has context `my-portal`, then your adapter configuration is loaded from the `$FUSE_HOME/etc/my-portal-keycloak.json` file.

\
