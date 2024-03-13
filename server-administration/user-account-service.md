# User Account Service

Keycloak has a built-in User Account Service which every user has access to. This service allows users to manage their account, change their credentials, update their profile, and view their login sessions. The URL to this service is `<server-root>/auth/realms/{realm-name}/account`.

Account Service

![account-service-profile.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/account-service-profile.png)

The initial page is the user’s profile, which is the `Account` left menu item. This is where they specify basic data about themselves. This screen can be extended to allow the user to manage additional attributes. See the [Server Development](https://keycloak.gitbooks.io/documentation/content/server\_development/index.html) for more details.

The `Password` left menu item allows the user to change their password.

Password Update

![account-service-password.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/account-service-password.png)

The `Authenticator` menu item allows the user to set up OTP if they desire. This will only show up if OTP is a valid authentication mechanism for your realm. Users are given directions to install [FreeOTP](https://fedorahosted.org/freeotp/) or [Google Authenticator](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2) on their mobile device to be their OTP generator. The QR code you see in the screen shot can be scanned into the FreeOTP or Google Authenticator mobile application for nice and easy setup.

OTP Authenticator

![account-service-authenticator.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/account-service-authenticator.png)

The `Federated Identity` menu item allows the user to link their account with an [identity broker](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/identity-broker.html#\_identity\_broker) (this is usually used to link social provider accounts together). This will show the list of external identity providers you have configured for your realm.

Federated Identity

![account-service-federated-identity.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/account-service-federated-identity.png)

The `Sessions` menu item allows the user to view and manage which devices are logged in and from where. They can perform logout of these sessions from this screen too.

Sessions

![account-service-sessions.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/account-service-sessions.png)

The `Applications` menu item shows users which applications they have access to.

Applications

![account-service-apps.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/account-service-apps.png)

#### Themeable <a href="#themeable" id="themeable"></a>

Like all UIs in Keycloak, the User Account Service is completely themeable and internationalizable. See the [Server Development](https://keycloak.gitbooks.io/documentation/content/server\_development/index.html) for more details.
