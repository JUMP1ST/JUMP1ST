# Themes and Internationalization

Themes allow you to change the look and feel of any UI in Keycloak. Themes are configured per realm. To change a theme go to the `Realm Settings` left menu item and click on the `Themes` tab.

Themes Tab

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Pick the theme you want for each UI category and click `Save`.

Login Theme

Username password entry, OTP entry, new user registration, and other similar screens related to login.

Account Theme

Each user has an User Account Management UI.

Admin Console Theme

The skin of the Keycloak Admin Console.

Email Theme

Whenever Keycloak has to send out an email, it uses templates defined in this theme to craft the email.

The [Server Development](https://keycloak.gitbooks.io/documentation/content/server\_development/index.html) goes into how to create a new themes or modify existing ones.

**Internationalization**

Every UI screen is internationalized in Keycloak. The default language is English, but if you turn on the `Internationalization` switch on the `Theme` tab you can choose which locales you want to support and what the default locale will be. The next time a user logs in, they will be able to choose a language on the login page to use for the login screens, User Account Management UI, and Admin Console. The [Server Development](https://keycloak.gitbooks.io/documentation/content/server\_development/index.html) explains how you can offer additional languages.
