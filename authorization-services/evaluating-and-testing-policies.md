# Evaluating and Testing Policies

When designing your policies, you can simulate authorization requests to test how your policies are being evaluated.

You can access the Policy Evaluation Tool by clicking the `Evaluate` tab when editing a resource server. There you can specify different inputs to simulate real authorization requests and test the effect of your policies.

![Policy Evaluation Tool](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/policy-evaluation-tool/policy-evaluation-tool.png)

#### Providing Identity Information <a href="#providing_identity_information" id="providing_identity_information"></a>

The **Identity Information** filters can be used to specify the user requesting permissions.

You can also click **Entitlement** to obtain all permissions for the user you selected.

#### Providing Contextual Information <a href="#providing_contextual_information" id="providing_contextual_information"></a>

The **Contextual Information** filters can be used to define additional attributes to the evaluation context, so that policies can obtain these same attributes.

#### Providing the Permissions <a href="#providing_the_permissions" id="providing_the_permissions"></a>

The **Permissions** filters can be used to build an authorization request. You can request permissions for a set of one or more resources and scopes. If you want to simulate authorization requests based on all protected resources and scopes, click **Add** without specifying any `Resources` or `Scopes`.

When you’ve specified your desired values, click **Evaluate**.
