# Forgot Password

If you enable it, users are able to reset their credentials if they forget their password or lose their OTP generator. Go to the `Realm Settings` left menu item, and click on the `Login` tab. Switch on the `Forgot Password` switch.

Login Tab

![login-tab.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/login-tab.png)

A `forgot password` link will now show up on your login pages.

Forgot Password Link

![forgot-password-link.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/forgot-password-link.png)

Clicking on this link will bring the user to a page where they can enter in their username or email and receive an email with a link to reset their credentials.

Forgot Password Page

![forgot-password-page.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/forgot-password-page.png)

The text sent in the email is completely configurable. You just need to extend or edit the theme associated with it. See the [Server Development](https://keycloak.gitbooks.io/documentation/content/server\_development/index.html) for more information.

When the user clicks on the email link, they will be asked to update their password, and, if they have an OTP generator set up, they will also be asked to reconfigure this as well. Depending on the security requirements of your organization you may not want users to be able to reset their OTP generator through email. You can change this behavior by going to the `Authentication` left menu item, clicking on the `Flows` tab, and selecting the `Reset Credentials` flow:

Reset Credentials Flow

![reset-credentials-flow.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/reset-credentials-flow.png)

If you do not want OTP reset, then just chose the `disabled` radio button to the right of `Reset OTP`.
