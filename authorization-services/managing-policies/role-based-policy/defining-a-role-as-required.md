# Defining a Role as Required

When creating a role-based policy, you can specify a specific role as `Required`. When you do that, the policy will grant access only if the user requesting access has been granted **all** the **required** roles. Both realm and client roles can be configured as such.

Example of Required Role

![Example of Required Role](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/policy/create-role.png)

To specify a role as required, select the `Required` checkbox for the role you want to configure as required.

Required roles can be useful when your policy defines multiple roles but only a subset of them are mandatory. In this case, you can combine realm and client roles to enable an even more fine-grained role-based access control (RBAC) model for your application. For example, you can have policies specific for a client and require a specific client role associated with that client. Or you can enforce that access is granted only in the presence of a specific realm role. You can also combine both approaches within the same policy.

\
