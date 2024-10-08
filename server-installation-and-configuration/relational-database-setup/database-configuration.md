# Database Configuration

The configuration for this component is found in the `standalone.xml`, `standalone-ha.xml`, or `domain.xml` file in your distribution. The location of this file depends on your [operating mode](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/operating-mode.html#\_operating-mode).

Database Config

```xml
<subsystem xmlns="urn:jboss:domain:keycloak-server:1.1">
    ...
    <spi name="connectionsJpa">
     <provider name="default" enabled="true">
         <properties>
             <property name="dataSource" value="java:jboss/datasources/KeycloakDS"/>
             <property name="initializeEmpty" value="false"/>
             <property name="migrationStrategy" value="manual"/>
             <property name="migrationExport" value="${jboss.home.dir}/keycloak-database-update.sql"/>
         </properties>
     </provider>
    </spi>
    ...
</subsystem>
```

Possible configuration options are:

dataSource

JNDI name of the dataSource

jta

boolean property to specify if datasource is JTA capable

driverDialect

Value of database dialect. In most cases you don’t need to specify this property as dialect will be autodetected by Hibernate.

initializeEmpty

Initialize database if empty. If set to false the database has to be manually initialized. If you want to manually initialize the database set migrationStrategy to `manual` which will create a file with SQL commands to initialize the database. Defaults to true.

migrationStrategy

Strategy to use to migrate database. Valid values are `update`, `manual` and `validate`. Update will automatically migrate the database schema. Manual will export the required changes to a file with SQL commands that you can manually execute on the database. Validate will simply check if the database is up-to-date.

migrationExport

Path for where to write manual database initialization/migration file.

showSql

Specify whether Hibernate should show all SQL commands in the console (false by default). This is very verbose!

formatSql

Specify whether Hibernate should format SQL commands (true by default)

globalStatsInterval

Will log global statistics from Hibernate about executed DB queries and other things. Statistics are always reported to server log at specified interval (in seconds) and are cleared after each report.

schema

Specify the database schema to use

| Note | These configuration switches and more are described in the [_WildFly 10 Documentation_](https://docs.jboss.org/author/display/WFLY10/JPA+Reference+Guide#JPAReferenceGuide-Hibernateproperties). |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
