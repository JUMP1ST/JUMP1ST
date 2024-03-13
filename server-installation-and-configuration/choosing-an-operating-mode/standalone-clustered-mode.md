# Standalone Clustered Mode

Standalone clustered operation mode is for when you want to run Keycloak within a cluster. This mode requires that you have a copy of the Keycloak distribution on each machine you want to run a server instance. This mode can be very easy to deploy initially, but can become quite cumbersome. To make a configuration change you’ll have to modify each distribution on each machine. For a large cluster this can become time consuming and error prone.

**Standalone Clustered Configuration**

The distribution has a mostly pre-configured app server configuration file for running within a cluster. It has all the specific infrastructure settings for networking, databases, caches, and discovery. This file resides in _…​/standalone/configuration/standalone-ha.xml_. There’s a few things missing from this configuration. You can’t run Keycloak in a cluster without a configuring a shared database connection. You also need to deploy some type of load balancer in front of the cluster. The [clustering](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/clustering.html#\_clustering) and [database](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/database.html#\_database) sections of this guide walk you though these things.

Standalone HA Config

![standalone-ha-config-file.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/keycloak-images/standalone-ha-config-file.png)

| Warning | Any changes you make to this file while the server is running will not take effect and may even be overwritten by the server. Instead use the the command line scripting or the web console of Wildfly. See the [_WildFly 10 Documentation_](https://docs.jboss.org/author/display/WFLY10/Documentation) for more information. |
| ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |

**Standalone Clustered Boot Script**

You use the same boot scripts to start Keycloak as you do in standalone mode. The difference is that you pass in an additional flag to point to the HA config file.

Standalone Clustered Boot Scripts

![standalone-boot-files.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/keycloak-images/standalone-boot-files.png)

To boot the server:

Linux/Unix

```
$ .../bin/standalone.sh --server-config=standalone-ha.xml
```

Windows

```
> ...\bin\standalone.bat --server-config=standalone-ha.xml
```
