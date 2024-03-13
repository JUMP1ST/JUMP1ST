# Revocation Policies

If your system is compromised you will want a way to revoke all sessions and access tokens that have been handed out. You can do this by going to the `Revocation` tab of the `Sessions` screen.

Revocation

![revocation.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/revocation.png)

You can only set a time-based revocation policy. The console allows you to specify a time and date where any session or token issued before that time and date is invalid. The `Set to now` will set the policy to the current time and date. The `Push` button will push this revocation policy to any registered OIDC client that has the Keycloak OIDC client adapter installed.
