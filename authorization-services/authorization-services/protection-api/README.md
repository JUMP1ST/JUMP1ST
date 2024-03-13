# Protection API

The Protection API provides a UMA-compliant set of endpoints providing:

*   **Resource Registration**

    With this endpoint, resource servers can manage their resources remotely and enable [policy enforcers](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/enforcer/overview.html#\_enforcer\_overview) to query the server for the resources that need protection.
*   **Permission Registration**

    In the UMA protocol, resource servers access this endpoint, which issues permission tickets.

An important requirement for this API is that _only_ resource servers are allowed to access its endpoints using a special OAuth2 access token called a protection API token (PAT). In UMA, a PAT is a token with the scope **uma\_protection**.
