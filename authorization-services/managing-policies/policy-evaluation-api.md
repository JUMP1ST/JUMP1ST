# Policy Evaluation API

When writing rule-based policies using JavaScript or JBoss Drools, Keycloak provides an Evaluation API that provides useful information to help determine whether a permission should be granted.

This API consists of a few interfaces that provides you access to information such as:

* The permission being requested
* The identity that is requesting the permission, from which you can obtain claims/attributes
* Runtime environment and any other attribute associated with the execution context

The main interface is **org.keycloak.authorization.policy.evaluation.Evaluation**, which defines the following contract:

```java
public interface Evaluation {

    /**
     * Returns the {@link ResourcePermission} to be evaluated.
     *
     * @return the permission to be evaluated
     */
    ResourcePermission getPermission();

    /**
     * Returns the {@link EvaluationContext}. Which provides access to the whole evaluation runtime context.
     *
     * @return the evaluation context
     */
    EvaluationContext getContext();

    /**
     * Grants the requested permission to the caller.
     */
    void grant();

    /**
     * Denies the requested permission.
     */
    void deny();
}
```

When processing an authorization request, Keycloak creates an `Evaluation` instance before evaluating any policy. This instance is then passed to each policy to determine whether access is **GRANT** or **DENY**.

Policies determine this by invoking the `grant()` or `deny()` methods on an `Evaluation` instance. By default, the state of the `Evaluation` instance is denied, which means that your policies must explicitly invoke the `grant()` method to indicate to the policy evaluation engine that permission should be granted.

For more information about the Evaluation API see the [JavaDocs](http://www.keycloak.org/docs/javadocs/index.html).

**The Evaluation Context**

The evaluation context provides useful information to policies during their evaluation.

```java
public interface EvaluationContext {

    /**
     * Returns the {@link Identity} that represents an entity (person or non-person) to which the permissions must be granted, or not.
     *
     * @return the identity to which the permissions must be granted, or not
     */
    Identity getIdentity();

    /**
     * Returns all attributes within the current execution and runtime environment.
     *
     * @return the attributes within the current execution and runtime environment
     */
    Attributes getAttributes();
}
```

From this interface, policies can obtain:

* The authenticated `Identity`
* Information about the execution context and runtime environment

The `Identity` is built based on the OAuth2 Access Token that was sent along with the authorization request, and this construct has access to all claims extracted from the original token. For example, if you are using a _Protocol Mapper_ to include a custom claim in a OAuth2 Access Token you can also access this claim from a policy and use it to build your conditions.

The `EvaluationContext` also gives you access to attributes related to both the execution and runtime environments. For now, there only a few built-in attributes.

| Name                          | Description                               | Type                                 |
| ----------------------------- | ----------------------------------------- | ------------------------------------ |
| kc.time.date\_time            | Current date and time                     | String. Format `MM/dd/yyyy hh:mm:ss` |
| kc.client.network.ip\_address | IPv4 address of the client                | String                               |
| kc.client.network.host        | Client’s host name                        | String                               |
| kc.client.id                  | The client id                             | String                               |
| kc.client.user\_agent         | The value of the 'User-Agent' HTTP header | String\[]                            |
| kc.realm.name                 | The name of the realm                     | String                               |
