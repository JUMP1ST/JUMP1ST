# Installing Distribution Files

The Keycloak Server has three downloadable distributions:

* 'keycloak-SNAPSHOT.\[zip|tar.gz]'
* 'keycloak-overlay-SNAPSHOT.\[zip|tar.gz]'
* 'keycloak-demo-SNAPSHOT.\[zip|tar.gz]'

The 'keycloak-SNAPSHOT.\[zip|tar.gz]' file is the server only distribution. It contains nothing other than the scripts and binaries to run the Keycloak Server. To unpack this file just run your operating system’s `unzip` or `gunzip` and `tar` utilities.

The 'keycloak-overlay-SNAPSHOT.\[zip|tar.gz]' file is a Wildfly add-on that allows you to install Keycloak Server on top of an existing WildFly distribution. We do not support users that want to run their applications and Keycloak on the same server instance. To install the Keycloak Service Pack, just unzip it in the root directory of your WildFly distribution, open the bin directory in a shell and run `./jboss-cli.[sh|bat] --file=keycloak-install.cli`.

The 'keycloak-demo-SNAPSHOT.\[zip|tar.gz]' contains the server binaries, all documentation and all examples. It is preconfigured with both the OIDC and SAML client application adapters and can deploy any of the distribution examples out of the box with no configuration. This distribution is only recommended for those that want to test drive Keycloak. We do not support users that run the demo distribution in production.

To unpack of these files run the `unzip` or `gunzip` and `tar` utilities.
