# Creating Resources

Creating a resource is straightforward and generic. Your main concern is the granularity of the resources you create. In other words, resources can be created to represent a set of one or more resources and the way you define them is crucial to managing permissions.

To create a new resource, click **Create** in the right upper corner of the resource listing.

Add Resource

![Add Resource](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/resource/create.png)

In Keycloak, a resource defines a small set of information that is common to different types of resources, such as:

*   **Name**

    A human-readable and unique string describing this resource.
*   **Type**

    A string uniquely identifying the type of a set of one or more resources. The type is a _string_ used to group different resource instances. For example, the default type for the default resource that is automatically created is `urn:resource-server-name:resources:default`

**Typed Resources**

The type field of a resource can be used to group different resources together, so they can be protected using a common set of permissions.

**Resource Owners**

Resources also have an owner. By default, resources are owned by the resource server.

However, resources can also be associated with users, so you can create permissions based on the resource owner. For example, only the resource owner is allowed to delete or update a given resource.

**Managing Resources Remotely**

Resource management is also exposed through the [Protection API](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/service/protection/protection-api.html#\_service\_protection\_api) to allow resource servers to remotely manage their resources.

When using the Protection API, resource servers can be implemented to manage resources owned by their users. In this case, you can specify the user identifier to configure a resource as belonging to a specific user.

| Note | Keycloak provides resource servers complete control over their resources. In the future, we should be able to allow users to control their own resources as well as approve authorization requests and manage permissions, especially when using the UMA protocol. |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
