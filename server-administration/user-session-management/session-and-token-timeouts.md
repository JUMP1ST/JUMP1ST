# Session and Token Timeouts

Keycloak gives you fine grain control of session, cookie, and token timeouts. This is all done on the `Tokens` tab in the `Realm Settings` left menu item.

Tokens Tab

![tokens-tab.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/tokens-tab.png)

Let’s walk through each of the items on this page.

| Configuration                           | Description                                                                                                                                                                                                                                                                   |
| --------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Revoke Refresh Token                    | For OIDC clients that are doing the refresh token flow, this flag, if on, will revoke that refresh token and issue another with the request that the client has to use. This basically means that refresh tokens have a one time use.                                         |
| SSO Session Idle                        | Also pertains to OIDC clients. If the user is not active for longer than this timeout, the user session will be invalidated. How is idle time checked? A client requesting authentication will bump the idle timeout. Refresh token requests will also bump the idle timeout. |
| SSO Session Max                         | Maximum time before a user session is expired and invalidated. This is a hard number and time. It controls the maximum time a user session can remain active, regardless of activity.                                                                                         |
| Offline Session Idle                    | For [offline access](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/sessions/offline.html#\_offline-access), this is the time the session is allowed to remain idle before the offline token is revoked.                                   |
| Access Token Lifespan                   | When an OIDC access token is created, this value affects the expiration.                                                                                                                                                                                                      |
| Access Token Lifespan For Implicit Flow | With the Implicit Flow no refresh token is provided. For this reason there’s a separate timeout for access tokens created with the Implicit Flow.                                                                                                                             |
| Client login timeout                    | This is the maximum time that a client has to finish the Authorization Code Flow in OIDC.                                                                                                                                                                                     |
| Login timeout                           | Total time a login must take. If authentication takes longer than this time then the user will have to start the authentication process over.                                                                                                                                 |
| Login action timeout                    | Maximum time a user can spend on any one page in the authentication process.                                                                                                                                                                                                  |
| User-Initiated Action Lifespan          | Maximum time before an action permit sent by a user (e.g. forgot password e-mail) is expired. This value is recommended to be short because it is expected that the user would react to self-created action quickly.                                                          |
| Default Admin-Initiated Action Lifespan | Maximum time before an action permit sent to a user by an admin is expired. This value is recommended to be long to allow admins send e-mails for users that are currently offline. The default timeout can be overridden right before issuing the token.                     |
