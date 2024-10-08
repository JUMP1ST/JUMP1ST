# Admin Events

Any action an admin performs within the admin console can be recorded for auditing purposes. The Admin Console performs administrative functions by invoking on the Keycloak REST interface. Keycloak audits these REST invocations. The resulting events can then be viewed in the Admin Console.

To enable auditing of Admin actions, go to the `Events` left menu item and select the `Config` tab.

Event Configuration

![login-events-config.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/login-events-config.png)

In the `Admin Events Settings` section, turn on the `Save Events` switch.

Admin Event Configuration

![admin-events-settings.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/admin-events-settings.png)

The `Include Representation` switch will include any JSON document that is sent through the admin REST API. This allows you to view exactly what an admin has done, but can lead to a lot of information stored in the database. The `Clear admin events` button allows you to wipe out the current information stored.

To view the admin events go to the `Admin Events` tab.

Admin Events

![admin-events.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/admin-events.png)

If the `Details` column has a `Representation` box, you can click on that to view the JSON that was sent with that operation.

Admin Representation

![admin-events-representation.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/admin-events-representation.png)

You can also filter for the events you are interested in by clicking the `Filter` button.

Admin Event Filter

![admin-events-filter.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/admin-events-filter.png)
