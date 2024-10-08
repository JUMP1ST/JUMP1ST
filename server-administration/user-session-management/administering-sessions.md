# Administering Sessions

If you go to the `Sessions` left menu item you can see a top level view of the number of sessions that are currently active in the realm.

Sessions

![sessions.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/sessions.png)

A list of clients is given and how many active sessions there currently are for that client. You can also logout all users in the realm by clicking the `Logout all` button on the right side of this list.

**Logout All Limitations**

Any SSO cookies set will now be invalid and clients that request authentication in active browser sessions will now have to re-login. Only certain clients are notified of this logout event, specifically clients that are using the Keycloak OIDC client adapter. Other client types (i.e. SAML) will not receive a backchannel logout request.

It is important to note that any outstanding access tokens are not revoked by clicking `Logout all`. They have to expire naturally. You have to push a [revocation policy](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/sessions/revocation.html#\_revocation-policy) out to clients, but that also only works with clients using the Keycloak OIDC client adapter.

**Application Drilldown**

On the `Sessions` page, you can also drill down to each client. This will bring you to the `Sessions` tab of that client. Clicking on the `Show Sessions` button there allows you to see which users are logged into that application.

Application Sessions

![application-sessions.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/application-sessions.png)

**User Drilldown**

If you go to the `Sessions` tab of an individual user, you can also view the session information.

User Sessions

![user-sessions.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/user-sessions.png)
