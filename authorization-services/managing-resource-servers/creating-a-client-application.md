# Creating a Client Application

The first step to enable Keycloak is to create the client application that you want to turn into a resource server.

To create a client application, complete the following steps:

1.  Click **Clients**.

    Clients

    ![Clients](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/resource-server/client-list.png)
2.  On this page, click **Create**.

    Create Client

    ![Create Client](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/resource-server/client-create.png)
3. Type the `Client ID` of the client. For example, _my-resource-server_.
4.  Type the `Root URL` for your application. For example:

    ```bash
    http://${host}:${port}/my-resource-server
    ```
5.  Click **Save**. The client is created and the client Settings page opens. A page similar to the following is displayed:

    Client Settings

    ![Client Settings](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/resource-server/client-enable-authz.png)

\
