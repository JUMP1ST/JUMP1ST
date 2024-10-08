# SAML Entity Descriptors

Instead of manually registering a SAML 2.0 client, you can import it via a standard SAML Entity Descriptor XML file. There is an `Import` option on the Add Client page.

Add Client

![add-client-saml.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/add-client-saml.png)

Click the `Select File` button and load your entity descriptor file. You should review all the information there to make sure everything is set up correctly.

Some SAML client adapters like _mod-auth-mellon_ need the XML Entity Descriptor for the IDP. You can obtain this by going to this public URL: `root/auth/realms/{realm}/protocol/saml/descriptor`
