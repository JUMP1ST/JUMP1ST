# Replication and Failover

The `sessions`, `authenticationSessions`, `offlineSessions` and `loginFailures` caches are the only caches that may perform replication. Entries are not replicated to every single node, but instead one or more nodes is chosen as an owner of that data. If a node is not the owner of a specific cache entry it queries the cluster to obtain it. What this means for failover is that if all the nodes that own a piece of data go down, that data is lost forever. By default, Keycloak only specifies one owner for data. So if that one node goes down that data is lost. This usually means that users will be logged out and will have to login again.

You can change the number of nodes that replicate a piece of data by change the `owners` attribute in the `distributed-cache` declaration.

owners

```xml
<subsystem xmlns="urn:jboss:domain:infinispan:4.0">
   <cache-container name="keycloak" jndi-name="infinispan/Keycloak">
       <distributed-cache name="sessions" mode="SYNC" owners="2"/>
...
```

Here we’ve changed it so at least two nodes will replicate one specific user login session.

| Tip | The number of owners recommended is really dependent on your deployment. If you do not care if users are logged out when a node goes down, then one owner is good enough and you will avoid replication. |
| --- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
