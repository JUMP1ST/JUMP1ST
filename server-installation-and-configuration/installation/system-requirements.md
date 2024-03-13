# System Requirements

These are the requirements to run the Keycloak authentication server:

* Can run on any operating system that runs Java
* Java 8 JDK
* zip or gzip and tar
* At least 512M of RAM
* At least 1G of diskspace
* A shared external database like Postgres, MySql, Oracle, etc. Keycloak requires an external shared database if you want to run in a cluster. Please see the [database configuration](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/database.html#\_database) section of this guide for more information.
* Network multicast support on your machine if you want to run in a cluster. Keycloak can be clustered without multicast, but this requires a bunch of configuration changes. Please see the [clustering](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/clustering.html#\_clustering) section of this guide for more information.
* On Linux, it is recommended to use `/dev/urandom` as a source of random data to prevent Keycloak hanging due to lack of available entropy, unless `/dev/random` usage is mandated by your security policy. To achieve that on Oracle JDK 8 and OpenJDK 8, set the `java.security.egd` system property on startup to `file:/dev/urandom`.
