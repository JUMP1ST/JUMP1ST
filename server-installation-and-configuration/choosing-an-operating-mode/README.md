# Choosing an Operating Mode

Before deploying Keycloak in a production environment you need to decide which type of operating mode you are going to use. Will you run Keycloak within a cluster? Do you want a centralized way to manage your server configurations? Your choice of operating mode effects how you configure databases, configure caching and even how you boot the server.

| Tip | The Keycloak is built on top of the Wildfly Application Server. This guide will only go over the basics for deployment within a specific mode. If you want specific information on this, a better place to go would be the [_WildFly 10 Documentation_](https://docs.jboss.org/author/display/WFLY10/Documentation). |
| --- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
