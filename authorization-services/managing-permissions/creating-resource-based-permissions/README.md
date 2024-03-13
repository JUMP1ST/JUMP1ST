# Creating Resource-Based Permissions

A resource-based permission defines a set of one or more resources to protect using a set of one or more authorization policies.

To create a new resource-based permission, select **Resource-based** in the dropdown list in the upper right corner of the permission listing.

Add Resource-Based Permission

![Add Resource-Based Permission](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/permission/create-resource.png)

**Configuration**

*   **Name**

    A human-readable and unique string describing the permission. A best practice is to use names that are closely related to your business and security requirements, so you can identify them more easily.
*   **Description**

    A string containing details about this permission.
*   **Apply To Resource Type**

    Specifies if the permission is applied to all resources with a given type. When selecting this field, you are prompted to enter the resource type to protect.

    *   Resource Type

        Defines the resource type to protect. When defined, this permission is evaluated for all resources matching that type.
*   **Resources**

    Defines a set of one or more resources to protect.
*   **Apply Policy**

    Defines a set of one or more policies to associate with a permission.
*   **Decision Strategy**

    The [Decision Strategy](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/permission/decision-strategy.html#\_permission\_decision\_strategies) for this permission.

\
