# RDBMS Setup Checklist

These are the steps you will need to perform to get an RDBMS configured for Keycloak.

1. Locate and download a JDBC driver for your database
2. Package the driver JAR into a module and install this module into the server
3. Declare the JDBC driver in the configuration profile of the server
4. Modify the datasource configuration to use your database’s JDBC driver
5. Modify the datasource configuration to define the connection parameters to your database

This chapter will use PostgresSQL for all its examples. Other databases follow the same steps for installation.
