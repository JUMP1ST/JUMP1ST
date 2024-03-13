# Realm Roles

Realm-level roles are a global namespace to define your roles. You can see the list of built-in and created roles by clicking the `Roles` left menu item.

![roles.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/roles.png)

To create a role, click **Add Role** on this page, enter in the name and description of the role, and click **Save**.

Add Role

![role.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/role.png)

The value for the `description` field is localizable by specifying a substitution variable with `${var-name}` strings. The localized value is then configured within property files in your theme. See the [Server Development](https://keycloak.gitbooks.io/documentation/content/server\_development/index.html) for more information on localization. If a client requires user _consent_, this description string is displayed on the consent page for the user.

If the client has to explicitly request for a realm role, set `Scope Param Required` to true. The role then has to be specified using the `scope` parameter when requesting a token. Multiple realm roles are separated by space:

`scope=admin user`
