# Rule-Based Policy

With this type of policy you can define conditions for your permissions using [Drools](http://www.drools.org/), which is a rule evaluation environment. It is one of the _Rule-Based_ policy types supported by Keycloak, and provides flexibility to write any policy based on the [Evaluation API](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/policy/evaluation-api.html#\_policy\_evaluation\_api).

To create a new Rule-based policy, in the dropdown list in the right upper corner of the policy listing, select **Rule**.

Add Rule Policy

![Add Rule Policy](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/policy/create-drools.png)

**Configuration**

*   **Name**

    A human-readable and unique string describing the policy. We strongly suggest that you use names that are closely related with your business and security requirements, so you can identify them more easily and also know what they actually mean.
*   **Description**

    A string with more details about this policy.
*   **Policy Maven Artifact**

    A Maven groupId-artifactId-version (GAV) pointing to an artifact where the rules are defined. Once you have provided the GAV, you can click **Resolve** to load both **Module** and **Session** fields.

    *   Group Id

        The groupId of the artifact.
    *   Artifact Id

        The artifactId of the artifact.
    *   Version

        The version of the artifact.
*   **Module**

    The module used by this policy. You must provide a module to select a specific session from which rules will be loaded.
*   **Session**

    The session used by this policy. The session provides all the rules to evaluate when processing the policy.
*   **Update Period**

    Specifies an interval for scanning for artifact updates.
*   **Logic**

    The [Logic](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/policy/logic.html#\_policy\_logic) of this policy to apply after the other conditions have been evaluated.

**Examples**

Here is a simple example of a Drools-based policy that uses attribute-based access control (ABAC) to define a condition that evaluates to a GRANT only if the authenticated user is the owner of the requested resource:

```javascript
import org.keycloak.authorization.policy.evaluation.Evaluation;
rule "Authorize Resource Owner"
    dialect "mvel"
    when
       $evaluation : Evaluation(
           $identity: context.identity,
           $permission: permission,
           $permission.resource != null && $permission.resource.owner.equals($identity.id)
       )
    then
        $evaluation.grant();
end
```

You can even use another variant of ABAC to obtain attributes from the identity and define a condition accordingly:

```javascript
import org.keycloak.authorization.policy.evaluation.Evaluation;
rule "Authorize Using Identity Information"
    dialect "mvel"
    when
       $evaluation : Evaluation(
           $identity: context.identity,
           identity.attributes.containsValue("someAttribute", "you_can_access")
       )
    then
        $evaluation.grant();
end
```

For more information about what you can access from the `org.keycloak.authorization.policy.evaluation.Evaluation` interface, see [Evaluation API](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/policy/evaluation-api.html#\_policy\_evaluation\_api).

\
