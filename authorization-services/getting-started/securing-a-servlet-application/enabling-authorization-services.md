# Enabling Authorization Services

You can enable authorization services in an existing client application configured to use the OpenID Connect Protocol. You can also create a new client.

To create a new client, complete the following steps:

1.  Click **Clients** to start creating a new client application and fill in the **Client ID**, **Client Protocol**, and **Root URL** fields.

    Create Client Application

    ![Create Client Application](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/getting-started/hello-world/create-client.png)
2.  Click **Save**. The Client Details page is displayed.

    Client Details

    ![Client Details](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/getting-started/hello-world/enable-authz.png)
3. On the Client Details page, click the **Authorization Enabled** switch to **ON**, and then click **Save**. A new **Authorization** tab is displayed for the client.
4.  Click the **Authorization** tab and an Authorization Settings page similar to the following is displayed:

    Authorization Settings

    ![Authorization Settings](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/getting-started/hello-world/authz-settings.png)

When you enable authorization services for a client application, Keycloak automatically creates several [default settings](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/resource-server/default-config.html#\_resource\_server\_default\_config) for your client authorization configuration.

For more information about authorization configuration, see [Enabling Authorization Services](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/resource-server/enable-authorization.html#\_resource\_server\_enable\_authorization).

\
