# Composite Roles

Any realm or client level role can be turned into a _composite role_. A _composite role_ is a role that has one or more additional roles associated with it. When a composite role is mapped to the user, the user also gains the roles associated with that composite. This inheritance is recursive so any composite of composites also gets inherited.

To turn a regular role into a composite role, go to the role detail page and flip the `Composite Role` switch on.

Composite Role

![composite-role.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/composite-role.png)

Once you flip this switch the role selection UI will be displayed lower on the page and you’ll be able to associate realm level and client level roles to the composite you are creating. In this example, the `employee` realm-level role was associated with the `developer` composite role. Any user with the `developer` role will now also inherit the `employee` role too.

| Note | When tokens and SAML assertions are created, any composite will also have its associated roles added to the claims and assertions of the authentication response sent back to the client. |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
