# Relational Database Setup

Keycloak comes with its own embedded Java-based relational database called H2. This is the default database that Keycloak will use to persist data and really only exists so that you can run the authentication server out of the box. We highly recommend that you replace it with a more production ready external database. The H2 database is not very viable in high concurrency situations and should not be used in a cluster either. The purpose of this chapter is to show you how to connect Keycloak to a more mature database.

Keycloak uses two layered technologies to persist its relational data. The bottom layered technology is JDBC. JDBC is a Java API that is used to connect to a RDBMS. There are different JDBC drivers per database type that are provided by your database vendor. This chapter discusses how to configure Keycloak to use one of these vendor-specific drivers.

The top layered technology for persistence is Hibernate JPA. This is a object to relational mapping API that maps Java Objects to relational data. Most deployments of Keycloak will never have to touch the configuration aspects of Hibernate, but we will discuss how that is done if you run into that rare circumstance.

| Note | Datasource configuration is covered much more thoroughly in [the datasource configuration chapter](https://docs.jboss.org/author/display/WFLY10/DataSource+configuration) in the _WildFly 10 Documentation_. |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
