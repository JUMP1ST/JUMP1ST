# Creating and Registering the Client

The next step you have to do is to define and register the client in the Keycloak Admin Console.

1. Log into the Admin Console with your admin account as you did in previous tutorials.
2.  In the top left dropdown menu select and manage the `demo` realm. Click `Clients` in the left side menu. The Clients page opens.

    Clients

    ![clients.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/getting\_started/keycloak-images/clients.png)
3. On the right click **Create**.
4.  Complete the fields as shown below:

    Add Client

    ![add-client.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/getting\_started/keycloak-images/add-client.png)
5.  After clicking the `Save` button your client application entry will be created. You now have to go back to the Wildfly instance that the application is deployed on and configure it so that this app is secured by Keycloak. You can obtain a template for the configuration you need by going to the `Installation` tab in the client entry in the Keycloak Admin Console.

    Installation Tab

    ![client-installation.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/getting\_started/keycloak-images/client-installation.png)
6.  Select **Keycloak OIDC JBoss Subsystem XML**. An XML template is generated that you’ll need to cut and paste.

    Template XML

    ![client-install-selected.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/getting\_started/keycloak-images/client-install-selected.png)
