# Modify the Keycloak Datasource

After declaring your JDBC driver, you have to modify the existing datasource configuration that Keycloak uses to connect it to your new external database. You’ll do this within the same configuration file and XML block that you registered your JDBC driver in. Here’s an example that sets up the connection to your new database:

Declare Your JDBC Drivers

```xml
  <subsystem xmlns="urn:jboss:domain:datasources:4.0">
     <datasources>
       ...
       <datasource jndi-name="java:jboss/datasources/KeycloakDS" pool-name="KeycloakDS" enabled="true" use-java-context="true">
           <connection-url>jdbc:postgresql://localhost/keycloak</connection-url>
           <driver>postgresql</driver>
           <pool>
               <max-pool-size>20</max-pool-size>
           </pool>
           <security>
               <user-name>William</user-name>
               <password>password</password>
           </security>
       </datasource>
        ...
     </datasources>
  </subsystem>
```

Search for the `datasource` definition for `KeycloakDS`. You’ll first need to modify the `connection-url`. The documentation for your vendor’s JDBC implementation should specify the format for this connection URL value.

Next define the `driver` you will use. This is the logical name of the JDBC driver you declared in the previous section of this chapter.

It is expensive to open a new connection to a database every time you want to perform a transaction. To compensate, the datasource implementation maintains a pool of open connections. The `max-pool-size` specifies the maximum number of connections it will pool. You may want to change the value of this depending on the load of your system.

Finally, with PostgreSQL at least, you need to define the database username and password that is needed to connect to the database. You may be worried that this is in clear text in the example. There are methods to obfuscate this, but this is beyond the scope of this guide.

| Note | For more information about datasource features, see [the datasource configuration chapter](https://docs.jboss.org/author/display/WFLY10/DataSource+configuration) in the _WildFly 10 Documentation_. |
| ---- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
