# Obtaining the Authorization Context

When policy enforcement is enabled, the permissions obtained from the server are available through `org.keycloak.AuthorizationContext`. This class provides several methods you can use to obtain permissions and ascertain whether a permission was granted for a particular resource or scope.

Obtaining the Authorization Context in a Servlet Container

```java
    HttpServletRequest request = ... // obtain javax.servlet.http.HttpServletRequest
    KeycloakSecurityContext keycloakSecurityContext =
        (KeycloakSecurityContext) request
            .getAttribute(KeycloakSecurityContext.class.getName());
    AuthorizationContext authzContext =
        keycloakSecurityContext.getAuthorizationContext();
```

| Note | For more details about how you can obtain a `KeycloakSecurityContext` consult the adapter configuration. The example above should be sufficient to obtain the context when running an application using any of the servlet containers supported by Keycloak. |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |

The authorization context helps give you more control over the decisions made and returned by the server. For example, you can use it to build a dynamic menu where items are hidden or shown depending on the permissions associated with a resource or scope.

```java
if (authzContext.hasResourcePermission("Project Resource")) {
    // user can access the Project Resource
}

if (authzContext.hasResourcePermission("Admin Resource")) {
    // user can access administration resources
}

if (authzContext.hasScopePermission("urn:project.com:project:create")) {
    // user can create new projects
}
```

The `AuthorizationContext` represents one of the main capabilities of Keycloak . From the examples above, you can see that the protected resource is not directly associated with the policies that govern them.

Consider some similar code using role-based access control (RBAC):

```java
if (User.hasRole('user')) {
    // user can access the Project Resource
}

if (User.hasRole('admin')) {
    // user can access administration resources
}

if (User.hasRole('project-manager')) {
    // user can create new projects
}
```

Although both examples address the same requirements, they do so in different ways. In RBAC, roles only _implicitly_ define access for their resources. With Keycloak you gain the capability to create more manageable code that focuses directly on your resources whether you are using RBAC, attribute-based access control (ABAC), or any other BAC variant. Either you have the permission for a given resource or scope, or you don’t.

Now, suppose your security requirements have changed and in addition to project managers, PMOs can also create new projects.

Security requirements change, but with Keycloak there is no need to change your application code to address the new requirements. Once your application is based on the resource and scope identifier, you need only change the configuration of the permissions or policies associated with a particular resource in the authorization server. In this case, the permissions and policies associated with the `Project Resource` and/or the scope `urn:project.com:project:create` would be changed.

**Using the AuthorizationContext to obtain an Authorization Client Instance**

The `AuthorizationContext` can also be used to obtain a reference to the [Authorization Client API](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/service/client-api.html#\_service\_client\_api) configured to your application:

```java
    ClientAuthorizationContext clientContext = ClientAuthorizationContext.class.cast(authzContext);
    AuthzClient authzClient = clientContext.getClient();
```

In some cases, resource servers protected by the policy enforcer need to access the APIs provided by the authorization server. With an `AuthzClient` instance in hands, resource servers can interact with the server in order to create resources or check for specific permissions programmatically.

\
