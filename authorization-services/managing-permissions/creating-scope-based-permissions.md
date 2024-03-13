# Creating Scope-Based Permissions

A scope-based permission defines a set of one or more scopes to protect using a set of one or more authorization policies. Unlike resource-based permissions, you can use this permission type to create permissions not only for a resource, but also for the scopes associated with it, providing more granularity when defining the permissions that govern your resources and the actions that can be performed on them.

To create a new scope-based permission, select **Scope-based** in the dropdown list in the upper right corner of the permission listing.

Add Scope-Based Permission

![Add Scope-Based Permission](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/permission/create-scope.png)

**Configuration**

*   **Name**

    A human-readable and unique string describing the permission. A best practice is to use names that are closely related to your business and security requirements, so you can identify them more easily.
*   **Description**

    A string containing details about this permission.
*   **Resource**

    Restricts the scopes to those associated with the selected resource. If none is selected, all scopes are available.
*   **Scopes**

    Defines a set of one or more scopes to protect.
*   **Apply Policy**

    Defines a set of one or more policies to associate with a permission.
*   **Decision Strategy**

    The [Decision Strategy](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/permission/decision-strategy.html#\_permission\_decision\_strategies) for this permission.

\
