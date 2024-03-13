# User Storage Federation

Many companies have existing user databases that hold information about users and their passwords or other credentials. In may cases, it is just not possible to migrate off of those existing stores to a pure Keycloak deployment. Keycloak can federate existing external user databases. Out of the box we have support for LDAP and Active Directory. You can also code your own extension for any custom user databases you might have using our User Storage SPI.

The way it works is that when a user logs in, Keycloak will look into its own internal user store to find the user. If it can’t find it there it will iterate over every User Storage provider you have configured for the realm until it finds a match. Data from the external store is mapped into a common user model that is consumed by the Keycloak runtime. This common user model can then be mapped to OIDC token claims and SAML assertion attributes.

External user databases rarely have every piece of data need to support all the features that Keycloak has. In this case, the User Storage Provider can opt to store some things locally in the Keycloak user store. Some providers even import the user locally and sync periodically with the external store. All this depends on the capabilities of the provider and how its configured. For example, your external user store may not support OTP. Depending on the provider, this OTP support can be handled and stored by Keycloak

#### Adding a Provider <a href="#adding_a_provider" id="adding_a_provider"></a>

To add a storage provider go to the `User Federation` left menu item in the Admin Console.

User Federation

![user-federation.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/user-federation.png)

On the right side, there is an `Add Provider` list box. Choose the provider type you want to add and you will be brought to the configuration page of that provider.
