# Examples

The Keycloak Authorization can also help you to quickly get started with the authorization services and understand how to apply the same concepts to your own applications.

If you are using the Keycloak Demo Distribution and you have it properly extracted in your file system:

* **keycloak-demo-SNAPSHOT.\[zip|tar.gz]**

you can check out the available examples from the following directory:

```bash
cd ${KEYCLOAK_DEMO_SERVER_DIR}/examples/authz
```

Or you can get them from [GitHub](https://github.com/keycloak/keycloak/tree/SNAPSHOT/examples/authz).

Each example has a `README` file with instructions about how to build, deploy, and test the example.

Here is a brief description of each example:

| Name                                                                                                                     | Description                                                                                                                                                                 |
| ------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [hello-world-authz-service](https://github.com/keycloak/keycloak/tree/SNAPSHOT/examples/authz/hello-world-authz-service) | A single-page application that is protected by a policy enforcer that decides whether a user can access that page based on the permissions obtained from a Keycloak Server. |
| [photoz](https://github.com/keycloak/keycloak/tree/SNAPSHOT/examples/authz/photoz)                                       | A simple application based on HTML5+AngularJS+JAX-RS that demonstrates how to enable fine-grained permissions to RESTFul based services and HTML5 clients.                  |
| [servlet-authz](https://github.com/keycloak/keycloak/tree/SNAPSHOT/examples/authz/servlet-authz)                         | A simple Servlet-based application that demonstrates how to enable fine-grained authorization to a JBoss/Wildfly Servlet Application.                                       |

\
