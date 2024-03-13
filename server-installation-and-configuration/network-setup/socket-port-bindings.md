# Socket Port Bindings

The ports opened for each socket have a pre-defined default that can be overriden at the command line or within configuration. To illustrate this configuration, let’s pretend you are running in [standalone mode](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/operating-mode/standalone.html#%3Cem%3Estandalone-mode) and open up the \_…​/standalone/configuration/standalone.xml. Search for `socket-binding-group`.

```xml
    <socket-binding-group name="standard-sockets" default-interface="public" port-offset="${jboss.socket.binding.port-offset:0}">
        <socket-binding name="management-http" interface="management" port="${jboss.management.http.port:9990}"/>
        <socket-binding name="management-https" interface="management" port="${jboss.management.https.port:9993}"/>
        <socket-binding name="ajp" port="${jboss.ajp.port:8009}"/>
        <socket-binding name="http" port="${jboss.http.port:8080}"/>
        <socket-binding name="https" port="${jboss.https.port:8443}"/>
        <socket-binding name="txn-recovery-environment" port="4712"/>
        <socket-binding name="txn-status-manager" port="4713"/>
        <outbound-socket-binding name="mail-smtp">
            <remote-destination host="localhost" port="25"/>
        </outbound-socket-binding>
    </socket-binding-group>
```

`socket-bindings` define socket connections that will be opened by the server. These bindings specify the `interface` (bind address) they use as well as what port number they will open. The ones you will be most interested in are:

http

Defines the port used for Keycloak HTTP connections

https

Defines the port used for Keycloak HTTPS connections

ajp

This socket binding defines the port used for the AJP protocol. This protocol is used by Apache HTTPD server in conjunction `mod-cluster` when you are using Apache HTTPD as a load balancer.

management-http

Defines the HTTP connection used by Wildfly CLI and web console.

When running in [domain mode](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/operating-mode/domain.html#%3Cem%3Edomain-mode) setting the socket configurations is a bit trickier as the example \_domain.xml file has multiple `socket-binding-groups` defined. If you scroll down to the `server-group` definitions you can see what `socket-binding-group` is used for each `server-group`.

domain socket bindings

```xml
    <server-groups>
        <server-group name="load-balancer-group" profile="load-balancer">
            ...
            <socket-binding-group ref="load-balancer-sockets"/>
        </server-group>
        <server-group name="auth-server-group" profile="auth-server-clustered">
            ...
            <socket-binding-group ref="ha-sockets"/>
        </server-group>
    </server-groups>
```

| Note | There are many more options available when setting up `socket-binding-group` definitions. For more information, see [the socket binding group](https://docs.jboss.org/author/display/WFLY10/Interfaces+and+ports) in the _WildFly 10 Documentation_. |
| ---- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
