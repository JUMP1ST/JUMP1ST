# Setting Up a Load Balancer or Proxy

This section discusses a number of things you need to configure before you can put a reverse proxy or load balancer in front of your clustered Keycloak deployment. It also covers configuring the built in load balancer that was [Clustered Domain Example](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/operating-mode/domain.html#\_clustered-domain-example).

**Identifying Client IP Addresses**

A few features in Keycloak rely on the fact that the remote address of the HTTP client connecting to the authentication server is the real IP address of the client machine. Examples include:

* Event logs - a failed login attempt would be logged with the wrong source IP address
* SSL required - if the SSL required is set to external (the default) it should require SSL for all external requests
* Authentication flows - a custom authentication flow that uses the IP address to for example show OTP only for external requests
* Dynamic Client Registration

This can be problematic when you have a reverse proxy or loadbalancer in front of your Keycloak authentication server. The usual setup is that you have a frontend proxy sitting on a public network that load balances and forwards requests to backend Keycloak server instances located in a private network. There is some extra configuration you have to do in this scenario so that the actual client IP address is forwarded to and processed by the Keycloak server instances. Specifically:

* Configure your reverse proxy or loadbalancer to properly set `X-Forwarded-For` and `X-Forwarded-Proto` HTTP headers.
* Configure your reverse proxy or loadbalancer to preserve the original 'Host' HTTP header.
* Configure the authentication server to read the client’s IP address from `X-Forwarded-For header`.

Configuring your proxy to generate the `X-Forwarded-For` and `X-Forwarded-Proto` HTTP headers and preserving the original `Host` HTTP header is beyond the scope of this guide. Take extra precautions to ensure that the `X-Forwared-For` header is set by your proxy. If your proxy isn’t configured correctly, then _rogue_ clients can set this header themselves and trick Keycloak into thinking the client is connecting from a different IP address than it actually is. This becomes really important if you are doing any black or white listing of IP addresses.

Beyond the proxy itself, there are a few things you need to configure on the Keycloak side of things. If your proxy is forwarding requests via the HTTP protocol, then you need to configure Keycloak to pull the client’s IP address from the `X-Forwarded-For` header rather than from the network packet. To do this, open up the profile configuration file (_standalone.xml_, _standalone-ha.xml_, or _domain.xml_ depending on your [operating mode](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/operating-mode.html#\_operating-mode)) and look for the `urn:jboss:domain:undertow:3.0` XML block.

`X-Forwarded-For` HTTP Config

```xml
<subsystem xmlns="urn:jboss:domain:undertow:3.0">
   <buffer-cache name="default"/>
   <server name="default-server">
      <ajp-listener name="ajp" socket-binding="ajp"/>
      <http-listener name="default" socket-binding="http" redirect-socket="https"
          proxy-address-forwarding="true"/>
      ...
   </server>
   ...
</subsystem>
```

Add the `proxy-address-forwarding` attribute to the `http-listener` element. Set the value to `true`.

If your proxy is using the AJP protocol instead of HTTP to forward requests (i.e. Apache HTTPD + mod-cluster), then you have to configure things a little differently. Instead of modifying the `http-listener`, you need to add a filter to pull this information from the AJP packets.

`X-Forwarded-For` AJP Config

```xml
<subsystem xmlns="urn:jboss:domain:undertow:3.0">
     <buffer-cache name="default"/>
     <server name="default-server">
         <ajp-listener name="ajp" socket-binding="ajp"/>
         <http-listener name="default" socket-binding="http" redirect-socket="https"/>
         <host name="default-host" alias="localhost">
             ...
             <filter-ref name="proxy-peer"/>
         </host>
     </server>
        ...
     <filters>
         ...
         <filter name="proxy-peer"
                 class-name="io.undertow.server.handlers.ProxyPeerAddressHandler"
                 module="io.undertow.core" />
     </filters>
 </subsystem>
```

**Enable HTTPS/SSL with a Reverse Proxy**

Assuming that your reverse proxy doesn’t use port 8443 for SSL you also need to configure what port HTTPS traffic is redirected to.

```xml
<subsystem xmlns="urn:jboss:domain:undertow:3.0">
    ...
    <http-listener name="default" socket-binding="http"
        proxy-address-forwarding="true" redirect-socket="proxy-https"/>
    ...
</subsystem>
```

Add the `redirect-socket` attribute to the `http-listener` element. The value should be `proxy-https` which points to a socket binding you also need to define.

Then add a new `socket-binding` element to the `socket-binding-group` element:

```xml
<socket-binding-group name="standard-sockets" default-interface="public"
    port-offset="${jboss.socket.binding.port-offset:0}">
    ...
    <socket-binding name="proxy-https" port="443"/>
    ...
</socket-binding-group>
```

**Verify Configuration**

You can verify the reverse proxy or load balancer configuration by opening the path `/auth/realms/master/.well-known/openid-configuration` through the reverse proxy. For example if the reverse proxy address is `https://acme.com/` then open the URL `https://acme.com/auth/realms/master/.well-known/openid-configuration`. This will show a JSON document listing a number of endpoints for Keycloak. Make sure the endpoints starts with the address (scheme, domain and port) of your reverse proxy or load balancer. By doing this you make sure that Keycloak is using the correct endpoint.

You should also verify that Keycloak sees the correct source IP address for requests. Do check this you can try to login to the admin console with an invalid username and/or password. This should show a warning in the server log something like this:

```
08:14:21,287 WARN  XNIO-1 task-45 [org.keycloak.events] type=LOGIN_ERROR, realmId=master, clientId=security-admin-console, userId=8f20d7ba-4974-4811-a695-242c8fbd1bf8, ipAddress=X.X.X.X, error=invalid_user_credentials, auth_method=openid-connect, auth_type=code, redirect_uri=http://localhost:8080/auth/admin/master/console/?redirect_fragment=%2Frealms%2Fmaster%2Fevents-settings, code_id=a3d48b67-a439-4546-b992-e93311d6493e, username=admin
```

Check that the value of `ipAddress` is the IP address of the machine you tried to login with and not the IP address of the reverse proxy or load balancer.

**Using the Built-In Load Balancer**

This section covers configuring the built in load balancer that is discussed in the [Clustered Domain Example](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/operating-mode/domain.html#\_clustered-domain-example).

The [Clustered Domain Example](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/operating-mode/domain.html#\_clustered-domain-example) is only designed to run on one machine. To bring up a slave on another host, you’ll need to

1. Edit the _domain.xml_ file to point to your new host slave
2. Copy the server distribution. You don’t need the _domain.xml_, _host.xml_, or _host-master.xml_ files. Nor do you need the _standalone/_ directory.
3. Edit the _host-slave.xml_ file to change the bind addresses used or override them on the command line

**Register a New Host With Load Balancer**

Let’s look first at registering the new host slave with the load balancer configuration in _domain.xml_. Open this file and go to the undertow configuration in the `load-balancer` profile. Add a new `host` definition called `remote-host3` within the `reverse-proxy` XML block.

domain.xml reverse-proxy config

```xml
<subsystem xmlns="urn:jboss:domain:undertow:3.0">
  ...
  <handlers>
      <reverse-proxy name="lb-handler">
         <host name="host1" outbound-socket-binding="remote-host1" scheme="ajp" path="/" instance-id="myroute1"/>
         <host name="host2" outbound-socket-binding="remote-host2" scheme="ajp" path="/" instance-id="myroute2"/>
         <host name="remote-host3" outbound-socket-binding="remote-host3" scheme="ajp" path="/" instance-id="myroute3"/>
      </reverse-proxy>
  </handlers>
  ...
</subsystem>
```

The `output-socket-binding` is a logical name pointing to a `socket-binding` configured later in the _domain.xml_ file. the `instance-id` attribute must also be unique to the new host as this value is used by a cookie to enable sticky sessions when load balancing.

Next go down to the `load-balancer-sockets` `socket-binding-group` and add the `outbound-socket-binding` for `remote-host3`. This new binding needs to point to the host and port of the new host.

domain.xml outbound-socket-binding

```xml
<socket-binding-group name="load-balancer-sockets" default-interface="public">
    ...
    <outbound-socket-binding name="remote-host1">
        <remote-destination host="localhost" port="8159"/>
    </outbound-socket-binding>
    <outbound-socket-binding name="remote-host2">
        <remote-destination host="localhost" port="8259"/>
    </outbound-socket-binding>
    <outbound-socket-binding name="remote-host3">
        <remote-destination host="192.168.0.5" port="8259"/>
    </outbound-socket-binding>
</socket-binding-group>
```

**Master Bind Addresses**

Next thing you’ll have to do is to change the `public` and `management` bind addresses for the master host. Either edit the _domain.xml_ file as discussed in the [Bind Addresses](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/network/bind-address.html#\_bind-address) chapter or specify these bind addresses on the command line as follows:

```
$ domain.sh --host-config=host-master.xml -Djboss.bind.address=192.168.0.2 -Djboss.bind.address.management=192.168.0.2
```

**Host Slave Bind Addresses**

Next you’ll have to change the `public`, `management`, and domain controller bind addresses (`jboss.domain.master-address`). Either edit the _host-slave.xml_ file or specify them on the command line as follows:

```
$ domain.sh --host-config=host-slave.xml
     -Djboss.bind.address=192.168.0.5
      -Djboss.bind.address.management=192.168.0.5
       -Djboss.domain.master.address=192.168.0.2
```

The values of `jboss.bind.address` and `jboss.bind.addres.management` pertain to the host slave’s IP address. The value of `jboss.domain.master.address` need to be the IP address of the domain controller which is the management address of the master host.

**Configuring Other Load Balancers**

See [the load balancing](https://docs.jboss.org/author/display/WFLY10/High+Availability+Guide) section in the _WildFly 10 Documentation_ for information how to use other software-based load balancers.
