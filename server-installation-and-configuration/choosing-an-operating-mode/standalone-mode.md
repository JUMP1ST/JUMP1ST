# Standalone Mode

Standalone operating mode is only useful when you want to run one, and only one Keycloak server instance. It is not usable for clustered deployments and all caches are non-distributed and local-only. It is not recommended that you use standalone mode in production as you will have a single point of failure. If your standalone mode server goes down, users will not be able to log in. This mode is really only useful to test drive and play with the features of Keycloak

**Standalone Boot Script**

When running the server in standalone mode, there is a specific script you need to run to boot the server depending on your operating system. These scripts live in the _bin/_ directory of the server distribution.

Standalone Boot Scripts

![standalone-boot-files.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/keycloak-images/standalone-boot-files.png)

To boot the server:

Linux/Unix

```
$ .../bin/standalone.sh
```

Windows

```
> ...\bin\standalone.bat
```

**Standalone Configuration**

The bulk of this guide walks you through how to configure infrastructure level aspects of Keycloak. These aspects are configured in a configuration file that is specific to the application server that Keycloak is a derivative of. In the standalone operation mode, this file lives in _…​/standalone/configuration/standalone.xml_. This file is also used to configure non-infrastructure level things that are specific to Keycloak components.

Standalone Config File

![standalone-config-file.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/keycloak-images/standalone-config-file.png)

| Warning | Any changes you make to this file while the server is running will not take effect and may even be overwritten by the server. Instead use the the command line scripting or the web console of Wildfly. See the [_WildFly 10 Documentation_](https://docs.jboss.org/author/display/WFLY10/Documentation) for more information. |
| ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
