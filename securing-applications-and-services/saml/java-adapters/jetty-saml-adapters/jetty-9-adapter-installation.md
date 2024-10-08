# Jetty 9 Adapter Installation

Keycloak has a separate SAML adapter for Jetty 9.x. You then have to provide some extra configuration in each WAR you deploy to Jetty. Let’s go over these steps.

Adapters are no longer included with the appliance or war distribution.Each adapter is a separate download on the Keycloak download site. They are also available as a maven artifact.

You must unzip the Jetty 9.x distro into Jetty 9.x’s root directory. Including adapter’s jars within your WEB-INF/lib directory will not work!

```
$ cd $JETTY_HOME
$ unzip keycloak-saml-jetty92-adapter-dist.zip
```

Next, you will have to enable the keycloak module for your jetty.base.

```
$ cd your-base
$ java -jar $JETTY_HOME/start.jar --add-to-startd=keycloak
```
