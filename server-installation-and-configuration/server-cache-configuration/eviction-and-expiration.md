# Eviction and Expiration

There are multiple different caches configured for Keycloak. There is a realm cache that holds information about secured applications, general security data, and configuration options. There is also a user cache that contains user metadata. Both caches default to a maximum of 10000 entries and use a least recently used eviction strategy. Each of them is also tied to an object revisions cache that controls eviction in a clustered setup. This cache is created implicitely and has twice the configured size. There are also separate caches for user sessions, offline tokens, and login failures. These caches are unbounded in size as well.

The eviction policy and max entries for these caches can be configured in the _standalone.xml_, _standalone-ha.xml_, or _domain.xml_ depending on your [operating mode](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/operating-mode.html#\_operating-mode).

non-clustered

```xml
<subsystem xmlns="urn:jboss:domain:infinispan:4.0">
    <cache-container name="keycloak" jndi-name="infinispan/Keycloak">
        <local-cache name="realms">
            <eviction max-entries="10000" strategy="LRU"/>
        </local-cache>
        <local-cache name="users">
            <eviction max-entries="10000" strategy="LRU"/>
        </local-cache>
        <local-cache name="sessions"/>
        <local-cache name="offlineSessions"/>
        <local-cache name="loginFailures"/>
        <local-cache name="work"/>
        <local-cache name="authorization">
           <eviction strategy="LRU" max-entries="100"/>
        </local-cache>
        <local-cache name="keys">
            <eviction strategy="LRU" max-entries="1000"/>
            <expiration max-idle="3600000"/>
        </local-cache>
    </cache-container>
```

clustered

```xml
<subsystem xmlns="urn:jboss:domain:infinispan:4.0">
    <cache-container name="keycloak" jndi-name="infinispan/Keycloak">
        <transport lock-timeout="60000"/>
        <local-cache name="realms">
            <eviction max-entries="10000" strategy="LRU"/>
        </local-cache>
        <local-cache name="users">
            <eviction max-entries="10000" strategy="LRU"/>
        </local-cache>
        <distributed-cache name="sessions" mode="SYNC" owners="1"/>
        <distributed-cache name="offlineSessions" mode="SYNC" owners="1"/>
        <distributed-cache name="loginFailures" mode="SYNC" owners="1"/>
        <distributed-cache name="authorization" mode="SYNC" owners="1"/>
        <replicated-cache name="work" mode="SYNC"/>
        <local-cache name="keys">
            <eviction max-entries="1000" strategy="LRU"/>
            <expiration max-idle="3600000"/>
        </local-cache>
    </cache-container>
```

To limit or expand the number of allowed entries simply add or edit the `eviction` element or the `expiration` element of particular cache configuration.
