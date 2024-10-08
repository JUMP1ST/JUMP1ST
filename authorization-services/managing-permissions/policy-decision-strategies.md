# Policy Decision Strategies

When associating policies with a permission, you can also define a decision strategy to specify how to evaluate the outcome of the associated policies to determine access.

*   **Unanimous**

    The default strategy if none is provided. In this case, _all_ policies must evaluate to a positive decision for the final decision to be also positive.
*   **Affirmative**

    In this case, _at least one_ policy must evaluate to a positive decision for the final decision to be also positive.
*   **Consensus**

    In this case, the number of positive decisions must be greater than the number of negative decisions. If the number of positive and negative decisions is equal, the final decision will be negative.
