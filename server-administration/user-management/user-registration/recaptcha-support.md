# reCAPTCHA Support

To safeguard registration against bots, Keycloak has integration with Google reCAPTCHA. To enable this you need to first go to [Google Recaptcha Website](https://developers.google.com/recaptcha/) and create an API key so that you can get your reCAPTCHA site key and secret. (FYI, localhost works by default so you don’t have to specify a domain).

Next, there are a few steps you need to perform in the Keycloak Admin Console. Click the `Authentication` left menu item and go to the `Flows` tab. Select the `Registration` flow from the drop down list on this page.

Registration Flow

![registration-flow.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/registration-flow.png)

Set the 'reCAPTCHA' requirement to `Required` by clicking the appropriate radio button. This will enable reCAPTCHA on the screen. Next, you have to enter in the reCAPTCHA site key and secret that you generated at the Google reCAPTCHA Website. Click on the 'Actions' button that is to the right of the reCAPTCHA flow entry, then "Config" link, and enter in the reCAPTCHA site key and secret on this config page.

Recaptcha Config Page

![recaptcha-config.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/recaptcha-config.png)

The final step you have to do is to change some default HTTP response headers that Keycloak sets. Keycloak will prevent a website from including any login page within an iframe. This is to prevent clickjacking attacks. You need to authorize Google to use the registration page within an iframe. Go to the `Realm Settings` left menu item and then go to the `Security Defenses` tab. You will need to add `https://www.google.com` to the values of both the `X-Frame-Options` and `Content-Security-Policy` headers.

Authorizing Iframes

![security-headers.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/security-headers.png)

Once you do this, reCAPTCHA should show up on your registration page. You may want to edit _register.ftl_ in your login theme to muck around with the placement and styling of the reCAPTCHA button. See the [Server Development](https://keycloak.gitbooks.io/documentation/content/server\_development/index.html) for more information on extending and creating themes.
