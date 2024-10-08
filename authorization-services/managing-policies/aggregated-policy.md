# Aggregated Policy

As mentioned previously, Keycloak allows you to build a policy of policies, a concept referred to as policy aggregation. You can use policy aggregation to reuse existing policies to build more complex ones and keep your permissions even more decoupled from the policies that are evaluated during the processing of authorization requests.

To create a new aggregated policy, select **Aggregated** in the dropdown list located in the right upper corner of the policy listing.

Add an Aggregated Policy

![Add Aggregated Policy](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/policy/create-aggregated.png)

Let’s suppose you have a resource called _Confidential Resource_ that can be accessed only by users from the _keycloak.org_ domain and from a certain range of IP addresses. You can create a single policy with both conditions. However, you want to reuse the domain part of this policy to apply to permissions that operates regardless of the originating network.

You can create separate policies for both domain and network conditions and create a third policy based on the combination of these two policies. With an aggregated policy, you can freely combine other policies and then apply the new aggregated policy to any permission you want.

| Note | When creating aggregated policies, be mindful that you are not introducing a circular reference or dependency between policies. If a circular dependency is detected, you cannot create or update the policy. |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

**Configuration**

*   **Name**

    A human-readable and unique string describing the policy. We strongly suggest that you use names that are closely related with your business and security requirements, so you can identify them more easily and also know what they mean.
*   **Description**

    A string with more details about this policy.
*   **Apply Policy**

    Defines a set of one or more policies to associate with a policy.
*   **Decision Strategy**

    The decision strategy for this permission.
*   **Logic**

    The [Logic](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/policy/logic.html#\_policy\_logic) of this policy to apply after the other conditions have been evaluated.

**Decision Strategy for Aggregated Policies**

When creating aggregated policies, you can also define the decision strategy that will be used to determine the final decision based on the outcome from each policy.

*   **Unanimous**

    The default strategy if none is provided. In this case, _all_ policies must evaluate to a positive decision for the final decision to be also positive.
*   **Affirmative**

    In this case, _at least one_ policy must evaluate to a positive decision in order for the final decision to be also positive.
*   **Consensus**

    In this case, the number of positive decisions must be greater than the number of negative decisions. If the number of positive and negative decisions is the same, the final decision will be negative.
