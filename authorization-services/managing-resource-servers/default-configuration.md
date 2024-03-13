# Default Configuration

When you create a resource server, Keycloak creates a default configuration for your newly created resource server.

The default configuration consists of:

* A default protected resource representing all resources in your application.
* A policy that always grants access to the resources protected by this policy.
* A permission that governs access to all resources based on the default policy.

The default protected resource is referred to as the **default resource** and you can view it if you navigate to the **Resources** tab.

Default Resource

![Default Resource](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/resource-server/default-resource.png)

This resource defines a `Type`, namely `urn:my-resource-server:resources:default` and a `URI` `/*`. Here, the `URI` field defines a wildcard pattern that indicates to Keycloak that this resource represents all the paths in your application. In other words, when enabling [policy enforcement](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/enforcer/overview.html#\_enforcer\_overview) for your application, all the permissions associated with the resource will be examined before granting access.

The `Type` mentioned previously defines a value that can be used to create [typed resource permissions](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/permission/typed-resource-permission.html#\_permission\_typed\_resource) that must be applied to the default resource or any other resource you create using the same type.

The default policy is referred to as the **only from realm policy** and you can view it if you navigate to the **Policies** tab.

Default Policy

![Default Policy](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/resource-server/default-policy.png)

This policy is a [JavaScript-based policy](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/policy/js-policy.html#\_policy\_js) defining a condition that always grants access to the resources protected by this policy. If you click this policy you can see that it defines a rule as follows:

```js
// by default, grants any permission associated with this policy
$evaluation.grant();
```

Lastly, the default permission is referred to as the **default permission** and you can view it if you navigate to the **Permissions** tab.

Default Permission

![Default Permission](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/resource-server/default-permission.png)

This permission is a [resource-based permission](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/permission/create-resource.html#\_permission\_create\_resource), defining a set of one or more policies that are applied to all resources with a given type.

**Changing the Default Configuration**

You can change the default configuration by removing the default resource, policy, or permission definitions and creating your own.

| Note | The default configuration defines a resource that maps to all paths in your application. If you are about to write permissions to your own resources, be sure to remove the **Default Resource** or change its `URI` field to a more specific path in your application. Otherwise, the policy associated with the default resource (which by default always grants access) will allow Keycloak to grant access to any protected resource. |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
