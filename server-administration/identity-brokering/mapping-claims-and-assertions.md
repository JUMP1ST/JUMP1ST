# Mapping Claims and Assertions

You can import the SAML and OpenID Connect metadata provided by the external IDP you are authenticating with into the environment of the realm. This allows you to extract user profile metadata and other information so that you can make it available to your applications.

Each new user that logs into your realm via an external identity provider will have an entry for it created in the local Keycloak database. The act of importing metadata from the SAML or OIDC assertions and claims will create this data with the local realm database.

If you click on an identity provider listed in the `Identity Providers` page for your realm, you will be brought to the IDPs `Settings` tab. On this page is also a `Mappers` tab. Click on that tab to start mapping your incoming IDP metadata.

![identity-provider-mappers.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/identity-provider-mappers.png)

There is a `Create` button on this page. Clicking on this create button allows you to create a broker mapper. Broker mappers can import SAML attributes or OIDC ID/Access token claims into user attributes and user role mappings.

![identity-provider-mapper.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/identity-provider-mapper.png)

Select a mapper from the `Mapper Type` list. Hover over the tooltip to see a description of what the mapper does. The tooltips also describe what configuration information you need to enter. Click `Save` and your new mapper will be added.

For JSON based claims, you can use dot notation for nesting and square brackets to access array fields by index. For example 'contact.address\[0].country'.

To investigate the structure of user profile JSON data provided by social providers you can enable the `DEBUG` level logger `org.keycloak.social.user_profile_dump`. This is done in the server’s app-server configuration file (domain.xml or standalone.xml).
