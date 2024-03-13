# Impersonation

It is often useful for an admin to impersonate a user. For example, a user may be experiencing a bug in one of your applications and an admin may want to impersonate the user to see if they can duplicate the problem. Admins with the appropriate permission can impersonate a user. There are two locations an admin can initiate impersonation. The first is on the `Users` list tab.

Users

![user-search.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/user-search.png)

You can see here that the admin has searched for `jim`. Next to Jim’s account you can see an impersonate button. Click that to impersonate the user.

Also, you can impersonate the user from the user `Details` tab.

User Details

![user-details.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/user-details.png)

Near the bottom of the page you can see the `Impersonate` button. Click that to impersonate the user.

When impersonating, if the admin and the user are in the same realm, then the admin will be logged out and automatically logged in as the user being impersonated. If the admin and user are not in the same realm, the admin will remain logged in, but additionally be logged in as the user in that user’s realm. In both cases, the browser will be redirected to the impersonated user’s User Account Management page.

Any user with the realm’s `impersonation` role can impersonate a user. Please see the [Admin Console Access Control](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/admin-console-permissions.html#\_admin\_permissions) chapter for more details on assigning administration permissions.
