# Bind Addresses

By default Keycloak binds to the localhost loopback address `127.0.0.1`. That’s not a very useful default if you want the authentication server available on your network. Generally, what we recommend is that you deploy a reverse proxy or load balancer on a public network and route traffic to individual Keycloak server instances on a private network. In either case though, you still need to set up your network interfaces to bind to something other than `localhost`.

Setting the bind address is quite easy and can be done on the command line with either the _standalone.sh_ or _domain.sh_ boot scripts discussed in the [Choosing an Operating Mode](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/operating-mode.html#\_operating-mode) chapter.

```
$ standalone.sh -b 192.168.0.5
```

The `-b` switch sets the IP bind address for any public interfaces.

Alternatively, if you don’t want to set the bind address at the command line, you can edit the profile configuration of your deployment. Open up the profile configuration file (_standalone.xml_ or _domain.xml_ depending on your [operating mode](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/operating-mode.html#\_operating-mode)) and look for the `interfaces` XML block.

```xml
    <interfaces>
        <interface name="management">
            <inet-address value="${jboss.bind.address.management:127.0.0.1}"/>
        </interface>
        <interface name="public">
            <inet-address value="${jboss.bind.address:127.0.0.1}"/>
        </interface>
    </interfaces>
```

The `public` interface corresponds to subsystems creating sockets that are available publicly. An example of one of these subsystems is the web layer which serves up the authentication endpoints of Keycloak. The `management` interface corresponds to sockets opened up by the management layer of the Wildfly. Specifically the sockets which allow you to use the `jboss-cli.sh` command line interface and the Wildfly web console.

In looking at the `public` interface you see that it has a special string `${jboss.bind.address:127.0.0.1}`. This string denotes a value `127.0.0.1` that can be overriden on the command line by setting a Java system property, i.e.:

```
$ domain.sh -Djboss.bind.address=192.168.0.5
```

The `-b` is just a shorthand notation for this command. So, you can either change the bind address value directly in the profile config, or change it on the command line when you boot up.

| Note | There are many more options available when setting up `interface` definitions. For more information, see [the network interface](https://docs.jboss.org/author/display/WFLY10/Interfaces+and+ports) in the _WildFly 10 Documentation_. |
| ---- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
