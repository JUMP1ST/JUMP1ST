# Getting Started

Before you can use this tutorial, you need to complete the installation of Keycloak and create the initial admin user as shown in the [Getting Started Tutorial](https://keycloak.gitbooks.io/documentation/content/getting\_started/index.html) tutorial. There is one caveat to this. You have to run a separate instance on the same machine as Keycloak Server. This separate instance will run your Java Servlet application. Because of this you will have to run the Keycloak under a different port so that there are no port conflicts when running on the same machine. Use the `jboss.socket.binding.port-offset` system property on the command line. The value of this property is a number that will be added to the base value of every port opened by Keycloak Server.

To boot Keycloak Server:

Linux/Unix

```
$ .../bin/standalone.sh -Djboss.socket.binding.port-offset=100
```

Windows

```
> ...\bin\standalone.bat -Djboss.socket.binding.port-offset=100
```

For more details about how to install and configure a , please follow the steps on the [Securing Applications and Services Guide](https://keycloak.gitbooks.io/documentation/content/securing\_apps/index.html) tutorial.

After installing and booting both servers you should be able to access Keycloak Admin Console at [http://localhost:8180/auth/admin/](http://localhost:8180/auth/admin/) and also the instance at [http://localhost:8080](http://localhost:8080/).
