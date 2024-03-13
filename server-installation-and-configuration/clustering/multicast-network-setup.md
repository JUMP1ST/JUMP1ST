# Multicast Network Setup

Out of the box clustering support has a need to for IP Multicast. Multicast is a network broadcast protocol. This protocol is used at boot time to discover and join the cluster. It is also used to broadcast messages for the replication and invalidation distributed caches used by Keycloak.

The clustering subsystem for Keycloak runs on the JGroups stack. Out of the box, the bind addresses for clustering are bound to a private network interface with a default IP address of 127.0.0.1. You’ll have to edit your the _standalone-ha.xml_ or _domain.xml_ sections discussed in the [Bind Address](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/network/bind-address.html#\_bind-address) chapter.

private network config

```xml
    <interfaces>
        ...
        <interface name="private">
            <inet-address value="${jboss.bind.address.private:127.0.0.1}"/>
        </interface>
    </interfaces>
    <socket-binding-group name="standard-sockets" default-interface="public" port-offset="${jboss.socket.binding.port-offset:0}">
        ...
        <socket-binding name="jgroups-mping" interface="private" port="0" multicast-address="${jboss.default.multicast.address:230.0.0.4}" multicast-port="45700"/>
        <socket-binding name="jgroups-tcp" interface="private" port="7600"/>
        <socket-binding name="jgroups-tcp-fd" interface="private" port="57600"/>
        <socket-binding name="jgroups-udp" interface="private" port="55200" multicast-address="${jboss.default.multicast.address:230.0.0.4}" multicast-port="45688"/>
        <socket-binding name="jgroups-udp-fd" interface="private" port="54200"/>
        <socket-binding name="modcluster" port="0" multicast-address="224.0.1.105" multicast-port="23364"/>
        ...
    </socket-binding-group>
```

Things you’ll want to configure are the `jboss.bind.address.private` and `jboss.default.multicast.address` as well as the ports of the services on the clustering stack.

| Note | It is possible to cluster Keycloak without IP Multicast, but this topic is beyond the scope of this guide. For more information, see [JGroups](https://docs.jboss.org/author/display/WFLY10/JGroups+Subsystem) in the _WildFly 10 Documentation_. |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
