# Clustering

This section covers configuring Keycloak to run in a cluster. There’s a number of things you have to do when setting up a cluster, specifically:

* [Pick an operation mode](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/operating-mode.html#\_operating-mode)
* [Configure a shared external database](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/database.html#\_database)
* Set up a load balancer
* Supplying a private network that supports IP multicast

Picking an operation mode and configuring a shared database have been discussed earlier in this guide. In this chapter we’ll discuss setting up a load balancer and supplying a private network. We’ll also discuss some issues that you need to be aware of when booting up a host in the cluster.

| Note | It is possible to cluster Keycloak without IP Multicast, but this topic is beyond the scope of this guide. For more information, see [JGroups](https://docs.jboss.org/author/display/WFLY10/JGroups+Subsystem) chapter of the _WildFly 10 Documentation_. |
| ---- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
