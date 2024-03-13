# JBoss EAP/Wildfly Adapter

To be able to secure WAR apps deployed on JBoss EAP, WildFly or JBoss AS, you must install and configure the Keycloak adapter subsystem. You then have two options to secure your WARs.

You can provide an adapter config file in your WAR and change the auth-method to KEYCLOAK within web.xml.

Alternatively, you don’t have to modify your WAR at all and you can secure it via the Keycloak adapter subsystem configuration in `standalone.xml`. Both methods are described in this section.

**Installing the adapter**

Adapters are available as a separate archive depending on what server version you are using.

Install on Wildfly 9, 10 or 11:

```
$ cd $WILDFLY_HOME
$ unzip keycloak-wildfly-adapter-dist-SNAPSHOT.zip
```

Install on Wildfly 8:

```
$ cd $WILDFLY_HOME
$ unzip keycloak-wf8-adapter-dist-SNAPSHOT.zip
```

Install on JBoss EAP 7:

```
$ cd $EAP_HOME
$ unzip keycloak-eap7-adapter-dist-SNAPSHOT.zip
```

Install on JBoss EAP 6:

```
$ cd $EAP_HOME
$ unzip keycloak-eap6-adapter-dist-SNAPSHOT.zip
```

Install on JBoss AS 7.1:

```
$ cd $JBOSS_HOME
$ unzip keycloak-as7-adapter-dist-SNAPSHOT.zip
```

This ZIP archive contains JBoss Modules specific to the Keycloak adapter. It also contains JBoss CLI scripts to configure the adapter subsystem.

To configure the adapter subsystem if the server is not running execute:

Wildfly 11

```
$ ./bin/jboss-cli.sh --file=adapter-elytron-install-offline.cli
```

Any other server but Wildfly 11

```
$ ./bin/jboss-cli.sh --file=adapter-install-offline.cli
```

| Note | The offline script is not available for JBoss EAP 6 |
| ---- | --------------------------------------------------- |

Alternatively, if the server is running execute:

Wildfly 11

```
$ ./bin/jboss-cli.sh --file=adapter-elytron-install.cli
```

Any other server but Wildfly 11

```
$ ./bin/jboss-cli.sh --file=adapter-install.cli
```

**Required Per WAR Configuration**

This section describes how to secure a WAR directly by adding configuration and editing files within your WAR package.

The first thing you must do is create a `keycloak.json` adapter configuration file within the `WEB-INF` directory of your WAR.

The format of this configuration file is described in the [Java adapter configuration](https://wjw465150.gitbooks.io/keycloak-documentation/content/securing\_apps/topics/oidc/java/java-adapter-config.html#\_java\_adapter\_config) section.

Next you must set the `auth-method` to `KEYCLOAK` in `web.xml`. You also have to use standard servlet security to specify role-base constraints on your URLs.

Here’s an example:

```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
      version="3.0">

    <module-name>application</module-name>

    <security-constraint>
        <web-resource-collection>
            <web-resource-name>Admins</web-resource-name>
            <url-pattern>/admin/*</url-pattern>
        </web-resource-collection>
        <auth-constraint>
            <role-name>admin</role-name>
        </auth-constraint>
        <user-data-constraint>
            <transport-guarantee>CONFIDENTIAL</transport-guarantee>
        </user-data-constraint>
    </security-constraint>
    <security-constraint>
        <web-resource-collection>
            <web-resource-name>Customers</web-resource-name>
            <url-pattern>/customers/*</url-pattern>
        </web-resource-collection>
        <auth-constraint>
            <role-name>user</role-name>
        </auth-constraint>
        <user-data-constraint>
            <transport-guarantee>CONFIDENTIAL</transport-guarantee>
        </user-data-constraint>
    </security-constraint>

    <login-config>
        <auth-method>KEYCLOAK</auth-method>
        <realm-name>this is ignored currently</realm-name>
    </login-config>

    <security-role>
        <role-name>admin</role-name>
    </security-role>
    <security-role>
        <role-name>user</role-name>
    </security-role>
</web-app>
```

**Securing WARs via Adapter Subsystem**

You do not have to modify your WAR to secure it with Keycloak. Instead you can externally secure it via the Keycloak Adapter Subsystem. While you don’t have to specify KEYCLOAK as an `auth-method`, you still have to define the `security-constraints` in `web.xml`. You do not, however, have to create a `WEB-INF/keycloak.json` file. This metadata is instead defined within server configuration (i.e. `standalone.xml`) in the Keycloak subsystem definition.

```xml
<extensions>
  <extension module="org.keycloak.keycloak-adapter-subsystem"/>
</extensions>

<profile>
  <subsystem xmlns="urn:jboss:domain:keycloak:1.1">
     <secure-deployment name="WAR MODULE NAME.war">
        <realm>demo</realm>
        <auth-server-url>http://localhost:8081/auth</auth-server-url>
        <ssl-required>external</ssl-required>
        <resource>customer-portal</resource>
        <credential name="secret">password</credential>
     </secure-deployment>
  </subsystem>
</profile>
```

The `secure-deployment` `name` attribute identifies the WAR you want to secure. Its value is the `module-name` defined in `web.xml` with `.war` appended. The rest of the configuration corresponds pretty much one to one with the `keycloak.json` configuration options defined in [Java adapter configuration](https://wjw465150.gitbooks.io/keycloak-documentation/content/securing\_apps/topics/oidc/java/java-adapter-config.html#\_java\_adapter\_config).

The exception is the `credential` element.

To make it easier for you, you can go to the Keycloak Administration Console and go to the Client/Installation tab of the application this WAR is aligned with. It provides an example XML file you can cut and paste.

If you have multiple deployments secured by the same realm you can share the realm configuration in a separate element. For example:

```xml
<subsystem xmlns="urn:jboss:domain:keycloak:1.1">
    <realm name="demo">
        <auth-server-url>http://localhost:8080/auth</auth-server-url>
        <ssl-required>external</ssl-required>
    </realm>
    <secure-deployment name="customer-portal.war">
        <realm>demo</realm>
        <resource>customer-portal</resource>
        <credential name="secret">password</credential>
    </secure-deployment>
    <secure-deployment name="product-portal.war">
        <realm>demo</realm>
        <resource>product-portal</resource>
        <credential name="secret">password</credential>
    </secure-deployment>
    <secure-deployment name="database.war">
        <realm>demo</realm>
        <resource>database-service</resource>
        <bearer-only>true</bearer-only>
    </secure-deployment>
</subsystem>
```

**Security Domain**

To propagate the security context to the EJB tier you need to configure it to use the "keycloak" security domain. This can be achieved with the @SecurityDomain annotation:

```
import org.jboss.ejb3.annotation.SecurityDomain;
...

@Stateless
@SecurityDomain("keycloak")
public class CustomerService {

    @RolesAllowed("user")
    public List<String> getCustomers() {
        return db.getCustomers();
    }
}
```
