# Typed Resource Permission

Resource permissions can also be used to define policies that are to be applied to all resources with a given [type](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/resource/create.html#\_resource\_create\_type). This form of resource-based permission can be useful when you have resources sharing common access requirements and constraints.

Frequently, resources within an application can be categorized (or typed) based on the data they encapsulate or the functionality they provide. For example, a financial application can manage different banking accounts where each one belongs to a specific customer. Although they are different banking accounts, they share common security requirements and constraints that are globally defined by the banking organization. With typed resource permissions, you can define common policies to apply to all banking accounts, such as:

* Only the owner can manage his account
* Only allow access from the owner’s country and/or region
* Enforce a specific authentication method

To create a typed resource permission, click [Apply to Resource Type](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/permission/create-resource.html#\_permission\_create\_resource\_apply\_resource\_type) when creating a new resource-based permission. With `Apply to Resource Type` set to `On`, you can specify the type that you want to protect as well as the policies that are to be applied to govern access to all resources with type you have specified.

Example of a Typed Resource Permission

![Example of a Typed Resource Permission](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/permission/typed-resource-perm-example.png)

\
