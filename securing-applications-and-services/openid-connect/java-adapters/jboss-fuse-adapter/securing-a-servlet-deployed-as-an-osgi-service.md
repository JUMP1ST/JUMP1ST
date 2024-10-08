# Securing a Servlet Deployed as an OSGI Service

You can use this method if you have a servlet class inside your OSGI bundled project that is not deployed as a classic WAR application. Fuse uses [Pax Web Whiteboard Extender](https://ops4j1.jira.com/wiki/display/ops4j/Pax+Web+Extender+-+Whiteboard) to deploy such servlets as web applications.

To secure your servlet with Keycloak, complete the following steps:

1.  Keycloak provides PaxWebIntegrationService, which allows injecting jetty-web.xml and configuring security constraints for your application. You need to declare such services in the `OSGI-INF/blueprint/blueprint.xml` file inside your application. Note that your servlet needs to depend on it. An example configuration:

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <blueprint xmlns="http://www.osgi.org/xmlns/blueprint/v1.0.0"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xsi:schemaLocation="http://www.osgi.org/xmlns/blueprint/v1.0.0
               http://www.osgi.org/xmlns/blueprint/v1.0.0/blueprint.xsd">

        <!-- Using jetty bean just for the compatibility with other fuse services -->
        <bean id="servletConstraintMapping" class="org.eclipse.jetty.security.ConstraintMapping">
            <property name="constraint">
                <bean class="org.eclipse.jetty.util.security.Constraint">
                    <property name="name" value="cst1"/>
                    <property name="roles">
                        <list>
                            <value>user</value>
                        </list>
                    </property>
                    <property name="authenticate" value="true"/>
                    <property name="dataConstraint" value="0"/>
                </bean>
            </property>
            <property name="pathSpec" value="/product-portal/*"/>
        </bean>

        <bean id="keycloakPaxWebIntegration" class="org.keycloak.adapters.osgi.PaxWebIntegrationService"
              init-method="start" destroy-method="stop">
            <property name="jettyWebXmlLocation" value="/WEB-INF/jetty-web.xml" />
            <property name="bundleContext" ref="blueprintBundleContext" />
            <property name="constraintMappings">
                <list>
                    <ref component-id="servletConstraintMapping" />
                </list>
            </property>
        </bean>

        <bean id="productServlet" class="org.keycloak.example.ProductPortalServlet" depends-on="keycloakPaxWebIntegration">
        </bean>

        <service ref="productServlet" interface="javax.servlet.Servlet">
            <service-properties>
                <entry key="alias" value="/product-portal" />
                <entry key="servlet-name" value="ProductServlet" />
                <entry key="keycloak.config.file" value="/keycloak.json" />
            </service-properties>
        </service>

    </blueprint>
    ```

    * You might need to have the `WEB-INF` directory inside your project (even if your project is not a web application) and create the `/WEB-INF/jetty-web.xml` and `/WEB-INF/keycloak.json` files as in the [Classic WAR application](https://wjw465150.gitbooks.io/keycloak-documentation/content/securing\_apps/topics/oidc/java/fuse/classic-war.html#\_fuse\_adapter\_classic\_war) section. Note you don’t need the `web.xml` file as the security-constraints are declared in the blueprint configuration file.
2.  The `Import-Package` in `META-INF/MANIFEST.MF` must contain at least these imports:

    ```
    org.keycloak.adapters.jetty;version="SNAPSHOT",
    org.keycloak.adapters;version="SNAPSHOT",
    org.keycloak.constants;version="SNAPSHOT",
    org.keycloak.util;version="SNAPSHOT",
    org.keycloak.*;version="SNAPSHOT",
    *;resolution:=optional
    ```
