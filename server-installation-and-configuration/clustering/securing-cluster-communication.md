# Securing Cluster Communication

When cluster nodes are isolated on a private network it requires access to the private network to be able to join a cluster or to view communication in the cluster. In addition you can also enable authentication and encryption for cluster communication. As long as your private network is secure it is not necessary to enable authentication and encryption. Keycloak does not send very sensitive information on the cluster in either case.

If you want to enable authentication and encryption for clustering communication see [Securing a Cluster](https://access.redhat.com/documentation/en-us/red\_hat\_jboss\_enterprise\_application\_platform/7.0/html/configuration\_guide/configuring\_high\_availability#securing\_cluster) in the _JBoss EAP Configuration Guide_.
