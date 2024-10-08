# Troubleshooting

Note that when you run cluster, you should see message similar to this in the log of both cluster nodes:

```
INFO  [org.infinispan.remoting.transport.jgroups.JGroupsTransport] (Incoming-10,shared=udp)
ISPN000094: Received new cluster view: [node1/keycloak|1] (2) [node1/keycloak, node2/keycloak]
```

If you see just one node mentioned, it’s possible that your cluster hosts are not joined together.

Usually it’s best practice to have your cluster nodes on private network without firewall for communication among them. Firewall could be enabled just on public access point to your network instead. If for some reason you still need to have firewall enabled on cluster nodes, you will need to open some ports. Default values are UDP port 55200 and multicast port 45688 with multicast address 230.0.0.4. Note that you may need more ports opened if you want to enable additional features like diagnostics for your JGroups stack. Keycloak delegates most of the clustering work to Infinispan/JGroups. For more information, see [JGroups](https://docs.jboss.org/author/display/WFLY10/JGroups+Subsystem) in the _WildFly 10 Documentation_.
