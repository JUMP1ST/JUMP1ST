# Before You Start

Before you can participate in this tutorial, you need to complete the installation of Keycloak and create the initial admin user as shown in the [Installing and Booting](https://wjw465150.gitbooks.io/keycloak-documentation/content/getting\_started/topics/first-boot.html#\_install-boot) tutorial. There is one caveat to this. You have to run a separate Wildfly instance on the same machine as the Keycloak server. This separate instance will run your Java Servlet application. Because of this you will have to run the Keycloak under a different port so that there are no port conflicts when running on the same machine. Use the `jboss.socket.binding.port-offset` system property on the command line. The value of this property is a number that will be added to the base value of every port opened by the Keycloak server.

To boot the Keycloak server:

Linux/Unix

```
$ .../bin/standalone.sh -Djboss.socket.binding.port-offset=100
```

After booting up Keycloak, you can then access the admin console at [http://localhost:8180/auth/admin/](http://localhost:8180/auth/admin/)
