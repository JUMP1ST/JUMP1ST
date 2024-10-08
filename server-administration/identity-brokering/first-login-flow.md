# First Login Flow

When a user logs in through identity brokering some aspects of the user are imported and linked within the realm’s local database. When Keycloak successfully authenticates users through an external identity provider there can be two situations:

* There is already a Keycloak user account imported and linked with the authenticated identity provider account. In this case, Keycloak will just authenticate as the existing user and redirect back to application.
* There is not yet an existing Keycloak user account imported and linked for this external user. Usually you just want to register and import the new account into Keycloak database, but what if there is an existing Keycloak account with the same email? Automatically linking the existing local account to the external identity provider is a potential security hole as you can’t always trust the information you get from the external identity provider.

Different organizations have different requirements when dealing with some of the conflicts and situations listed above. For this, there is a `First Login Flow` option in the IDP settings which allows you to choose a [workflow](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/authentication/flows.html#\_authentication-flows) that will be used after a user logs in from an external IDP the first time. By default it points to `first broker login` flow, but you can configure and use your own flow and use different flows for different identity providers.

The flow itself is configured in admin console under `Authentication` tab. When you choose `First Broker Login` flow, you will see what authenticators are used by default. You can re-configure the existing flow. (For example you can disable some authenticators, mark some of them as `required`, configure some authenticators, etc).

**Default First Login Flow**

Let’s describe the default behaviour provided by `First Broker Login` flow.

Review Profile

This authenticator might display the profile info page, where the user can review his profile retrieved from an identity provider. The authenticator is configurable. You can set the `Update Profile On First Login` option. When `On`, users will be always presented with the profile page asking for additional information in order to federate their identities. When `missing`, users will be presented with the profile page only if some mandatory information (email, first name, last name) is not provided by the identity provider. If `Off`, the profile page won’t be displayed, unless user clicks in later phase on `Review profile info` link (page displayed in later phase by `Confirm Link Existing Account` authenticator)

Create User If Unique

This authenticator checks if there is already an existing Keycloak account with same email or username like the account from the identity provider. If it’s not, then the authenticator just creates a new local Keycloak account and links it with the identity provider and the whole flow is finished. Otherwise it goes to the next `Handle Existing Account` subflow. If you always want to ensure that there is no duplicated account, you can mark this authenticator as `REQUIRED` . In this case, the user will see the error page if there is existing Keycloak account and the user will need to link his identity provider account through Account management.

Confirm Link Existing Account

On the info page, the user will see that there is an existing Keycloak account with same email. He can review his profile again and use different email or username (flow is restarted and goes back to `Review Profile` authenticator). Or he can confirm that he wants to link the identity provider account with his existing Keycloak account. Disable this authenticator if you don’t want users to see this confirmation page, but go straight to linking identity provider account by email verification or re-authentication.

Verify Existing Account By Email

This authenticator is `ALTERNATIVE` by default, so it’s used only if the realm has SMTP setup configured. It will send mail to the user, where he can confirm that he wants to link the identity provider with his Keycloak account. Disable this if you don’t want to confirm linking by email, but instead you always want users to reauthenticate with their password (and alternatively OTP).

Verify Existing Account By Re-authentication

This authenticator is used if email authenticator is disabled or non-available (SMTP not configured for realm). It will display a login screen where the user needs to authenticate with his password to link his Keycloak account with the Identity provider. User can also re-authenticate with some different identity provider, which is already linked to his Keycloak account. You can also force users to use OTP. Otherwise it’s optional and used only if OTP is already set for the user account.
