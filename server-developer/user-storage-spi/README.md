# User Storage SPI

You can use the User Storage SPI to write extensions to Keycloak to connect to external user databases and credential stores. The built-in LDAP and ActiveDirectory support is an implementation of this SPI in action. Out of the box, Keycloak uses its local database to create, update, and look up users and validation credentials. Often though, organizations have existing external proprietary user databases that they cannot migrate to Keycloak’s data model. For those situations, application developers can write implementations of the User Storage SPI to bridge the external user store and the internal user object model that Keycloak uses to log in users and manage them.

When the Keycloak runtime needs to look up a user, such as when a user is logging in, it performs a number of steps to locate the user. It first looks to see if the user is in the user cache; if the user is found it uses that in-memory representation. Then it looks for the user within the Keycloak local database. If the user is not found, it then loops through User Storage SPI provider implementations to perform the user query until one of them returns the user the runtime is looking for. The provider queries the external user store for the user and maps the external data representation of the user to Keycloak’s user metamodel.

User Storage SPI provider implementations can also perform complex criteria queries, perform CRUD operations on users, validate and manage credentials, or perform bulk updates of many users at once. It depends on the capabilities of the external store.

User Storage SPI provider implementations are packaged and deployed similarly to (and often are) Java EE components. They are not enabled by default, but instead must be enabled and configured per realm under the `User Federation` tab in the administration console.

\
