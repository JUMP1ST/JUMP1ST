# General Configuration

The identity broker configuration is all based on identity providers. Identity providers are created for each realm and by default they are enabled for every single application. That means that users from a realm can use any of the registered identity providers when signing in to an application.

In order to create an identity provider click the `Identity Providers` left menu item.

Identity Providers

![identity-providers.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/identity-providers.png)

In the drop down list box, choose the identity provider you want to add. This will bring you to the configuration page for that identity provider type.

Add Identity Provider

![add-identity-provider.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/add-identity-provider.png)

Above is an example of configuring a Google social login provider. Once you configure an IDP, it will appear on the Keycloak login page as an option.

IDP login page

![identity-provider-login-page.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/identity-provider-login-page.png)

Social

Social providers allow you to enable social authentication in your realm. Keycloak makes it easy to let users log in to your application using an existing account with a social network. Currently Facebook, Google, Twitter, GitHub, LinkedIn, Microsoft, and StackOverflow are supported with more planned for the future.

Protocol-based

Protocol-based providers are those that rely on a specific protocol in order to authenticate and authorize users. They allow you to connect to any identity provider compliant with a specific protocol. Keycloak provides support for SAML v2.0 and OpenID Connect v1.0 protocols. It makes it easy to configure and broker any identity provider based on these open standards.

Although each type of identity provider has its own configuration options, all of them share some very common configuration. Regardless the identity provider you are creating, you’ll see the following configuration options available:

| Configuration          | Description                                                                                                                                                                                                                                                                                                                                                                                                          |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Alias                  | The alias is an unique identifier for an identity provider. It is used to reference an identity provider internally. Some protocols such as OpenID Connect require a redirect URI or callback url in order to communicate with an identity provider. In this case, the alias is used to build the redirect URI. Every single identity provider must have an alias. Examples are facebook, google, idp.acme.com, etc. |
| Enabled                | Turn the provider on/off                                                                                                                                                                                                                                                                                                                                                                                             |
| Hide On Login Page     | When this switch is on, this provider will not be shown as a login option on the login page. Clients can still request to use this provider by using the 'kc\_idp\_hint' parameter in the URL they use to request a login.                                                                                                                                                                                           |
| Link Only              | When this switch is on, this provider cannot be used to login users and will not be shown as an option on the login page. Existing accounts can still be linked with this provider though.                                                                                                                                                                                                                           |
| Store Tokens           | Whether or not to store the token received from the identity provider.                                                                                                                                                                                                                                                                                                                                               |
| Stored Tokens Readable | Whether or not users are allowed to retrieve the stored identity provider token. This also applies to the _broker_ client-level role _read token_                                                                                                                                                                                                                                                                    |
| Trust email            | If the identity provider supplies an email address this email address will be trusted. If the realm required email validation, users that log in from this IDP will not have to go through the email verification process.                                                                                                                                                                                           |
| GUI order              | The order number that sorts how the available IDPs are listed on the Keycloak login page.                                                                                                                                                                                                                                                                                                                            |
| First Login Flow       | This is the authentication flow that will be triggered for users that log into Keycloak through this IDP for the first time ever.                                                                                                                                                                                                                                                                                    |
| Post Login Flow        | Authentication flow that is triggered after the user finishes logging in with the external identity provider.                                                                                                                                                                                                                                                                                                        |
