# User Role Mappings

User role mappings can be assigned individually to each user through the `Role Mappings` tab for that single user.

Role Mappings

![user-role-mappings.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/user-role-mappings.png)

In the above example, we are about to assign the composite role `developer` that was created in the [Composite Roles](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/roles/composite.html#\_composite-roles) chapter.

Effective Role Mappings

![effective-role-mappings.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/effective-role-mappings.png)

Once the `developer` role is assigned, you see that the `employee` role that is associated with the `developer` composite shows up in the `Effective Roles`. `Effective Roles` are all roles that are explicitly assigned to the user as well as any roles that are inherited from composites.
