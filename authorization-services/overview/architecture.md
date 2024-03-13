# Architecture

![Keycloak AuthZ Architecture Overview](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/images/authz-arch-overview.png)

From a design perspective, is based on a well-defined set of authorization patterns providing these capabilities:

*   **Policy Administration Point (PAP)**

    Provides a set of UIs based on the Keycloak Administration Console to manage resource servers, resources, scopes, permissions, and policies. Part of this is also accomplished remotely through the use of the [Protection API](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/service/protection/protection-api.html#\_service\_protection\_api).
*   **Policy Decision Point (PDP)**

    Provides a distributable policy decision point to where authorization requests are sent and policies are evaluated accordingly with the permissions being requested. Part of this is also accomplished remotely through the use of the [Authorization](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/service/authorization/authorization-api.html#\_service\_authorization\_api) and [Entitlement](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/service/entitlement/entitlement-api.html#\_service\_entitlement\_api) APIs.
*   **Policy Enforcement Point (PEP)**

    Provides implementations for different environments to actually enforce authorization decisions at the resource server side. Keycloak provides some built-in [Policy Enforcers](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/enforcer/overview.html#\_enforcer\_overview).
*   **Policy Information Point (PIP)**

    Being based on Keycloak Authentication Server, you can obtain attributes from identities and runtime environment during the evaluation of authorization policies.

**The Authorization Process**

Three main processes define the necessary steps to understand how to use Keycloak to enable fine-grained authorization to your applications:

* **Resource Management**
* **Permission and Policy Management**
* **Policy Enforcement**

**Resource Management**

**Resource Management** involves all the necessary steps to define what is being protected.

![Resource Management Overview](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/images/resource-mgmt-process.png)

First, you need to specify Keycloak what are you looking to protect, which usually represents a web application or a set of one or more services. For more information on resource servers see [Terminology](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/overview/terminology.html#\_overview\_terminology).

Resource servers are managed using the Keycloak Administration Console. There you can enable any registered client application as a resource server and start managing the resources and scopes you want to protect.

![Resource Server Overview](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/images/rs-r-scopes.png)

A resource can be a web page, a RESTFul resource, a file in your file system, an EJB, and so on. They can represent a group of resources (just like a Class in Java) or they can represent a single and specific resource.

For instance, you might have a _Bank Account_ resource that represents all banking accounts and use it to define the authorization policies that are common to all banking accounts. However, you might want to define specific policies for _Alice Account_ (a resource instance that belongs to a customer), where only the owner is allowed to access some information or perform an operation.

Resources can be managed using the Keycloak Administration Console or the [Protection API](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/service/protection/protection-api.html#\_service\_protection\_api). In the latter case, resource servers are able to manage their resources remotely.

Scopes usually represent the actions that can be performed on a resource, but they are not limited to that. You can also use scopes to represent one or more attributes within a resource.

**Permission and Policy Management**

Once you have defined your resource server and all the resources you want to protect, you must set up permissions and policies.

This process involves all the necessary steps to actually define the security and access requirements that govern your resources.

![Permission and Policy Management Overview](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/images/policy-mgmt-process.png)

Policies define the conditions that must be satisfied to access or perform operations on something (resource or scope), but they are not tied to what they are protecting. They are generic and can be reused to build permissions or even more complex policies.

For instance,to allow access to a group of resources only for users granted with a role "User Premium,"" you can use RBAC (Role-based Access Control).

Keycloak provides a few built-in policy types (and their respective policy providers) covering the most common access control mechanisms. You can even create policies based on rules written using JavaScript or JBoss Drools.

Once you have your policies defined, you can start defining your permissions. Permissions are coupled with the resource they are protecting. Here you specify what you want to protect (resource or scope) and the policies that must be satisfied to grant or deny permission.

**Policy Enforcement**

**Policy Enforcement** involves the necessary steps to actually enforce authorization decisions to a resource server. This is achieved by enabling a **Policy Enforcement Point** or PEP at the resource server that is capable of communicating with the authorization server, ask for authorization data and control access to protected resources based on the decisions and permissions returned by the server.

![PEP Overview](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/images/pep-pattern-diagram.png)

Keycloak provides some built-in [Policy Enforcers](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/enforcer/overview.html#\_enforcer\_overview) implementations that you can use to protect your applications depending on the platform they are running on.

**Authorization Services**

Authorization services consist of the following RESTFul APIs:

* **Protection API**
* **Authorization API**
* **Entitlement API**

Each of these services provides a specific API covering the different steps involved in the authorization process.

**Protection API**

The **Protection API** is a [UMA-compliant](https://docs.kantarainitiative.org/uma/rec-uma-core.html) endpoint providing a small set of operations for resource servers to help them manage their resources and scopes. Only resource servers are allowed to access this API, which also requires a **uma\_protection** scope.

The operations provided by the Protection API can be organized in two main groups:

* **Resource Management**
  * Create Resource
  * Delete Resource
  * Find by Id
  * Find All
  * Find with filters (for example, search by name, type, or URI)
* **Permission Management**
  * Issue Permission Tickets

| Note | By default, Remote Resource Management is enabled. You can change that using the Keycloak Administration Console and only allow resource management through the console. |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |

When using the UMA protocol, the issuance of Permission Tickets by the Protection API is an important part of the whole authorization process. As described in a subsequent section, they represent the permissions being requested by the client and that are sent to the server to obtain a final token with all permissions granted during the evaluation of the permissions and policies associated with the resources and scopes being requested.

For more information, see [Protection API](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/service/protection/protection-api.html#\_service\_protection\_api).

**Authorization API**

The Authorization API is also a [UMA-compliant](https://docs.kantarainitiative.org/uma/rec-uma-core.html) endpoint providing a single operation that exchanges an Access Token and [Permission Ticket](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/overview/terminology.html#\_overview\_terminology\_permission\_ticket) with a Requesting Party Token (RPT).

The RPT contains all permissions granted to a client and can be used to call a resource server to get access to its protected resources.

When requesting an RPT you can also provide a previously issued RPT. In this case, the resulting RPT will consist of the union of the permissions from the previous RPT and the new ones within a permission ticket.

![Authorization API Overview](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/images/authz-calls.png)

For more information, see [Authorization API](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/service/authorization/authorization-api.html#\_service\_authorization\_api).

**Entitlement API**

The Entitlement API provides a 1-legged protocol to issue RPTs. Unlike the Authorization API, the Entitlement API only expects an access token.

From this API you can obtain all the entitlements or permissions for a user (based on the resources managed by a given resource server) or just the entitlements for a set of one or more resources.

![Entitlement API Overview](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/images/entitlement-calls.png)

For more information see [Entitlement API](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/service/entitlement/entitlement-api.html#\_service\_entitlement\_api).
