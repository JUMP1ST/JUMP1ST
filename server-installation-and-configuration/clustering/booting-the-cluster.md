# Booting the Cluster

Booting Keycloak in a cluster depends on your [operating mode](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/operating-mode.html#\_operating-mode)

Standalone Mode

```
$ bin/standalone.sh --server-config=standalone-ha.xml
```

Domain Mode

```
$ bin/domain.sh --host-config=host-master.xml
$ bin/domain.sh --host-config=host-slave.xml
```
