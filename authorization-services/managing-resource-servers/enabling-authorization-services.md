# Enabling Authorization Services

To turn your OIDC Client Application into a resource server and enable fine-grained authorization, click the **Authorization Enabled** switch to **ON** and click **Save**.

Enabling Authorization Services

![Enabling Authorization Services](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/resource-server/client-enable-authz.png)

A new Authorization tab is displayed for this client. Click the **Authorization** tab and a page similar to the following is displayed:

Resource Server Settings

![Resource Server Settings](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/resource-server/authz-settings.png)

The Authorization tab contains additional sub-tabs covering the different steps that you must follow to actually protect your application’s resources. Each tab is covered separately by a specific topic in this documentation. But here is a quick description about each one:

*   **Settings**

    General settings for your resource server. For more details about this page see the [Resource Server Settings](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/resource-server/enable-authorization.html#resource\_server\_settings) section.
*   **Resource**

    From this page, you can manage your application’s [resources](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/resource/overview.html#\_resource\_overview).
*   **Scope**

    From this page, you can manage [scopes](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/resource/overview.html#\_resource\_overview).
*   **Policies**

    From this page, you can manage [authorization policies](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/policy/overview.html#\_policy\_overview) and define the conditions that must be met to grant a permission.
*   **Permissions**

    From this page, you can manage the [permissions](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/permission/overview.html#\_permission\_overview) for your protected resources and scopes by linking them with the policies you created.
*   **Evaluate**

    From this page, you can [simulate authorization requests](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/policy-evaluation-tool/overview.html#\_policy\_evaluation\_overview) and view the result of the evaluation of the permissions and authorization policies you have defined.

**Resource Server Settings**

On the Resource Server Settings page, you can configure the policy enforcement mode, allow remote resource management, and export the authorization configuration settings.

*   **Policy Enforcement Mode**

    Specifies how policies are enforced when processing authorization requests sent to the server.

    *   **Enforcing**

        (default mode) Requests are denied by default even when there is no policy associated with a given resource.
    *   **Permissive**

        Requests are allowed even when there is no policy associated with a given resource.
    *   **Disabled**

        Disables the evaluation of all policies and allows access to all resources.
*   **Allow Remote Resource Management**

    Specifies whether resources can be managed remotely by the resource server. If false, resources can be managed only from the administration console.
*   **Export Settings**

    You can export the authorization configuration settings to a JSON file. Click **Export** to display the complete JSON configuration for download. The configuration file contains everything defined for a resource server: protected resources, scopes, permissions, and policies.
