# Installing the Client Adapter

Download the Wildfly distribution and unzip it into a directory on your machine.

Next download the WildFly OpenID Connect adapter distribution from [keycloak.org](http://www.keycloak.org/downloads.html).

Unzip this file into the root directory of your Wildfly distribution.

Next perform the following actions:

WildFly 10 and Linux/Unix

```
$ cd bin
$ ./jboss-cli.sh --file=adapter-install-offline.cli
```

Wildfly 11 and Linux/Unix

```
$ cd bin
$ ./jboss-cli.sh --file=adapter-elytron-install-offline.cli
```

This script will make the appropriate edits to the _…​/standalone/configuration/standalone.xml_ file of your app server distribution. Finally, boot the application server.

Linux/Unix

```
$ .../bin/standalone.sh
```
