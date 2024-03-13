# Sticky sessions

Typical cluster deployment consists of the load balancer (reverse proxy) and 2 or more Keycloak servers on private network. For performance purposes, it may be useful if load balancer forwards all requests related to particular browser session to the same Keycloak backend node.

The reason is, that Keycloak is using infinispan distributed cache under the covers for save data related to current authentication session and user session. The Infinispan distributed caches are configured with one owner by default. That means that particular session is saved just on one cluster node and the other nodes need to lookup the session remotely if they want to access it.

For example if authentication session with ID `123` is saved in the infinispan cache on `node1`, and then `node2` needs to lookup this session, it needs to send the request to `node1` over the network to return the particular session entity.

It is beneficial if particular session entity is always available locally, which can be done with the help of sticky sessions. The workflow in the cluster environment with the public frontend load balancer and two backend Keycloak nodes can be like this:

* User sends initial request to see the Keycloak login screen
* This request is served by the frontend load balancer, which forwards it to some random node (eg. node1). Strictly said, the node doesn’t need to be random, but can be chosen according to some other criterias (client IP address etc). It all depends on the implementation and configuration of underlying load balancer (reverse proxy).
* Keycloak creates authentication session with random ID (eg. 123) and saves it to the Infinispan cache.
* Infinispan distributed cache assigns the primary owner of the session based on the hash of session ID. See [Infinispan documentation](http://infinispan.org/docs/8.2.x/user\_guide/user\_guide.html#distribution\_mode) for more details around this. Let’s assume that infinispan assigned `node2` to be the owner of this session.
* Keycloak creates the cookie `AUTH_SESSION_ID` with the format like `<session-id>.<owner-node-id>` . In our example case, it will be `123.node2` .
* Response is returned to the user with the Keycloak login screen and the AUTH\_SESSION\_ID cookie in the browser

From this point, it is beneficial if load balancer forwards all the next requests to the `node2` as this is the node, who is owner of the authentication session with ID `123` and hence Infinispan can lookup this session locally. After authentication is finished, the authentication session is converted to user session, which will be also saved on `node2` because it has same ID `123` .

The sticky session is not mandatory for the cluster setup, however it is good for performance for the reasons mentioned above. You need to configure your loadbalancer to sticky over the `AUTH_SESSION_ID` cookie. How exactly do this is dependent on your loadbalancer.

Some loadbalancers can be configured to add the route information by themselves instead of rely on the backend Keycloak node. However adding the route by the Keycloak is more clever as Keycloak knows, who is the owner of particular session entity and can route it to that node, which is not necessarily the local node. We plan to support the setup where the route info won’t be added to AUTH\_SESSION\_ID cookie by the Keycloak server, but instead by the load balancer. However adding the cookie by Keycloak would be still the preferred option.

**Example cluster setup with mod\_cluster**

In the example, we will use [Mod Cluster](http://mod-cluster.jboss.org/) as load balancer. One of the key features of mod cluster is, that there is not much configuration on the load balancer side. Instead it requires support on the backend node side. Backend nodes communicate with the load balancer through the dedicated protocol called MCMP and they notify loadbalancer about various events (eg. node joined or left cluster, new application was deployed etc).

Example setup will consist of the one Wildfly 10 load balancer node and two Keycloak nodes.

Clustering example require MULTICAST to be enabled on machine’s loopback network interface. This can be done by running the following commands under root privileges (on linux):

```
route add -net 224.0.0.0 netmask 240.0.0.0 dev lo
ifconfig lo multicast
```

**Load Balancer Configuration**

Unzip the Wildfly 10 server somewhere. Assumption is location `EAP_LB`

Edit `EAP_LB/standalone/configuration/standalone.xml` file. In the undertow subsystem add the mod\_cluster configuration under filters like this:

```xml
<subsystem xmlns="urn:jboss:domain:undertow:4.0">
 ...
 <filters>
  ...
  <mod-cluster name="modcluster" advertise-socket-binding="modcluster"
      advertise-frequency="${modcluster.advertise-frequency:2000}"
      management-socket-binding="http" enable-http2="true"/>
 </filters>
```

and `filter-ref` under `default-host` like this:

```xml
<host name="default-host" alias="localhost">
    ...
    <filter-ref name="modcluster"/>
</host>
```

Then under `socket-binding-group` add this group:

```xml
<socket-binding name="modcluster" port="0"
    multicast-address="${jboss.modcluster.multicast.address:224.0.1.105}"
    multicast-port="23364"/>
```

Save the file and run the server:

```
cd $WILDFLY_LB/bin
./standalone.sh
```

**Backend node configuration**

Unzip the Keycloak server distribution to some location. Assuming location is `RHSSO_NODE1` .

Edit `RHSSO_NODE1/standalone/configuration/standalone-ha.xml` and configure datasource against the shared database. See [Database chapter](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/database/checklist.html#\_rdbms-setup-checklist) for more details.

In the undertow subsystem, add the `session-config` under the `servlet-container` element:

```xml
<servlet-container name="default">
    <session-cookie name="AUTH_SESSION_ID" http-only="true" />
    ...
</servlet-container>
```

Then you can configure `proxy-address-forwarding` as described in the chapter [Load Balancer](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/clustering/load-balancer.html#\_setting-up-a-load-balancer-or-proxy) . Note that mod\_cluster uses AJP connector by default, so you need to configure that one.

That’s all as mod\_cluster is already configured.

The node name of the Keycloak can be detected automatically based on the hostname of current server. However for more fine grained control, it is recommended to use system property `jboss.node.name` to specify the node name directly. It is especially useful in case that you test with 2 backend nodes on same physical server etc. So you can run the startup command like this:

```
cd $RHSSO_NODE1
./standalone.sh -c standalone-ha.xml -Djboss.socket.binding.port-offset=100 -Djboss.node.name=node1
```

Configure the second backend server in same way and run with different port offset and node name.

```
cd $RHSSO_NODE2
./standalone.sh -c standalone-ha.xml -Djboss.socket.binding.port-offset=200 -Djboss.node.name=node2
```

Access the server on `http://localhost:8080/auth` . Creation of admin user is possible just from local address and without load balancer (proxy) access, so you first need to access backend node directly on `http://localhost:8180/auth` to create admin user.
