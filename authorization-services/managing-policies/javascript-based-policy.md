# JavaScript-Based Policy

You can use this type of policy to define conditions for your permissions using JavaScript. It is one of the rule-based policy types supported by Keycloak, and provides flexibility to write any policy based on the [Evaluation API](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/policy/evaluation-api.html#\_policy\_evaluation\_api).

To create a new JavaScript-based policy, select **JavaScript** in the dropdown list in the upper right corner of the policy listing.

Add JavaScript Policy

![Add JavaScript Policy](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/policy/create-js.png)

**Configuration**

*   **Name**

    A human-readable and unique string describing the policy. A best practice is to use names that are closely related to your business and security requirements, so you can identify them more easily.
*   **Description**

    A string containing details about this policy.
*   **Code**

    The JavaScript code providing the conditions for this policy.
*   **Logic**

    The [Logic](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/policy/logic.html#\_policy\_logic) of this policy to apply after the other conditions have been evaluated.

**Examples**

Here is a simple example of a JavaScript-based policy that uses attribute-based access control (ABAC) to define a condition based on an attribute obtained from the execution context:

```javascript
var context = $evaluation.getContext();
var contextAttributes = context.getAttributes();

if (contextAttributes.containsValue('kc.client.network.ip_address', '127.0.0.1')) {
    $evaluation.grant();
}
```

You can also use role-based access control (RBAC):

```javascript
var identity = $evaluation.getIdentity();

if (identity.hasRole('keycloak_user')) {
    $evaluation.grant();
}
```

Or a combination of several access control mechanisms:

```javascript
var context = $evaluation.getContext();
var identity = context.getIdentity();
var attributes = identity.getAttributes();
var email = attributes.getValue('email').asString(0);

if (identity.hasRole('admin') || email.endsWith('@keycloak.org')) {
    $evaluation.grant();
}
```

When writing your own rules, keep in mind that the **$evaluation** object is an object implementing **org.keycloak.authorization.policy.evaluation.Evaluation**. For more information about what you can access from this interface, see the [Evaluation API](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/policy/evaluation-api.html#\_policy\_evaluation\_api).

\
