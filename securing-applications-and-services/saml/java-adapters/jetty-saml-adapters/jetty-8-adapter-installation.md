# Jetty 8 Adapter Installation

Keycloak has a separate SAML adapter for Jetty 8.1.x that you will have to install into your Jetty installation. You then have to provide some extra configuration in each WAR you deploy to Jetty. Let’s go over these steps.

Adapters are no longer included with the appliance or war distribution.Each adapter is a separate download on the Keycloak download site. They are also available as a maven artifact.

You must unzip the Jetty 8.1.x distro into Jetty 8.1.x’s root directory. Including adapter’s jars within your WEB-INF/lib directory will not work!

```
$ cd $JETTY_HOME
$ unzip keycloak-saml-jetty81-adapter-dist.zip
```

Next, you will have to enable the keycloak option. Edit start.ini and add keycloak to the options

```
#===========================================================
# Start classpath OPTIONS.
# These control what classes are on the classpath
# for a full listing do
#   java -jar start.jar --list-options
#-----------------------------------------------------------
OPTIONS=Server,jsp,jmx,resources,websocket,ext,plus,annotations,keycloak
```
