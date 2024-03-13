# User Registration

You can enable Keycloak to allow user self registration. When enabled, the login page has a registration link the user can click on to create their new account. Enabling registration is pretty simple. Go to the `Realm Settings` left menu and click it. Then go to the `Login` tab. There is a `User Registration` switch on this tab. Turn it on, then click the `Save` button.

Login Tab

![login-tab.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/login-tab.png)

After you enable this setting, a `Register` link should show up on the login page.

Registration Link

![registration-link.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/registration-link.png)

Clicking on this link will bring the user to the registration page where they have to enter in some user profile information and a new password.

Registration Form

![registration-form.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/registration-form.png)

You can change the look and feel of the registration form as well as removing or adding additional fields that must be entered. See the [Server Development](https://keycloak.gitbooks.io/documentation/content/server\_development/index.html) for more information.
