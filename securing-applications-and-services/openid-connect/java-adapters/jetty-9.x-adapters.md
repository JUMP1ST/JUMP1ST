# Jetty 9.x Adapters

Keycloak has a separate adapter for Jetty 9.1.x, Jetty 9.2.x and Jetty 9.3.x that you will have to install into your Jetty installation. You then have to provide some extra configuration in each WAR you deploy to Jetty. Let’s go over these steps.

**Adapter Installation**

Adapters are no longer included with the appliance or war distribution.Each adapter is a separate download on the Keycloak download site. They are also available as a maven artifact.

You must unzip the Jetty 9.x distro into Jetty 9.x’s [base directory.](https://www.eclipse.org/jetty/documentation/current/startup-base-and-home.html) Including adapter’s jars within your WEB-INF/lib directory will not work! In the example below, the Jetty base is named `your-base`:

```
$ cd your-base
$ unzip keycloak-jetty93-adapter-dist-2.5.0.Final.zip
```

Next, you will have to enable the `keycloak` module for your Jetty base:

```
$ java -jar $JETTY_HOME/start.jar --add-to-startd=keycloak
```

**Required Per WAR Configuration**

This section describes how to secure a WAR directly by adding config and editing files within your WAR package.

The first thing you must do is create a `WEB-INF/jetty-web.xml` file in your WAR package. This is a Jetty specific config file and you must define a Keycloak specific authenticator within it.

```
<?xml version="1.0"?>
<!DOCTYPE Configure PUBLIC "-//Mort Bay Consulting//DTD Configure//EN" "http://www.eclipse.org/jetty/configure_9_0.dtd">
<Configure class="org.eclipse.jetty.webapp.WebAppContext">
    <Get name="securityHandler">
        <Set name="authenticator">
            <New class="org.keycloak.adapters.jetty.KeycloakJettyAuthenticator">
            </New>
        </Set>
    </Get>
</Configure>
```

Next you must create a `keycloak.json` adapter config file within the `WEB-INF` directory of your WAR.

The format of this config file is describe in the [Java adapter configuration](https://wjw465150.gitbooks.io/keycloak-documentation/content/securing\_apps/topics/oidc/java/java-adapter-config.html#\_java\_adapter\_config) section.

| Warning | The Jetty 9.1.x adapter will not be able to find the `keycloak.json` file. You will have to define all adapter settings within the `jetty-web.xml` file as described below. |
| ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

Instead of using keycloak.json, you can define everything within the `jetty-web.xml`. You’ll just have to figure out how the json settings match to the `org.keycloak.representations.adapters.config.AdapterConfig` class.

```
<?xml version="1.0"?>
<!DOCTYPE Configure PUBLIC "-//Mort Bay Consulting//DTD Configure//EN" "http://www.eclipse.org/jetty/configure_9_0.dtd">
<Configure class="org.eclipse.jetty.webapp.WebAppContext">
  <Get name="securityHandler">
    <Set name="authenticator">
        <New class="org.keycloak.adapters.jetty.KeycloakJettyAuthenticator">
            <Set name="adapterConfig">
                <New class="org.keycloak.representations.adapters.config.AdapterConfig">
                    <Set name="realm">tomcat</Set>
                    <Set name="resource">customer-portal</Set>
                    <Set name="authServerUrl">http://localhost:8081/auth</Set>
                    <Set name="sslRequired">external</Set>
                    <Set name="credentials">
                        <Map>
                            <Entry>
                                <Item>secret</Item>
                                <Item>password</Item>
                            </Entry>
                        </Map>
                    </Set>
                </New>
            </Set>
        </New>
    </Set>
  </Get>
</Configure>
```

You do not have to crack open your WAR to secure it with keycloak. Instead create the jetty-web.xml file in your webapps directory with the name of yourwar.xml. Jetty should pick it up. In this mode, you’ll have to declare keycloak.json configuration directly within the xml file.

Finally you must specify both a `login-config` and use standard servlet security to specify role-base constraints on your URLs. Here’s an example:

```
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
      version="3.0">

	<module-name>customer-portal</module-name>

    <security-constraint>
        <web-resource-collection>
            <web-resource-name>Customers</web-resource-name>
            <url-pattern>/*</url-pattern>
        </web-resource-collection>
        <auth-constraint>
            <role-name>user</role-name>
        </auth-constraint>
        <user-data-constraint>
            <transport-guarantee>CONFIDENTIAL</transport-guarantee>
        </user-data-constraint>
    </security-constraint>

    <login-config>
        <auth-method>BASIC</auth-method>
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
