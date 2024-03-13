# Compromised Access and Refresh Tokens

There are a few things you can do to mitigate access tokens and refresh tokens from being stolen. The most important thing is to enforce SSL/HTTPS communication between Keycloak and its clients and applications. It might seem obvious, but since Keycloak does not have SSL enabled by default, an administrator might not realize that it is necessary.

Another thing you can do to mitigate leaked access tokens is to shorten their lifespans. You can specify this within the [timeouts page](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/sessions/timeouts.html#\_timeouts). Short lifespans (minutes) for access tokens for clients and applications to refresh their access tokens after a short amount of time. If an admin detects a leak, they can logout all user sessions to invalidate these refresh tokens or set up a revocation policy. Making sure refresh tokens always stay private to the client and are never transmitted ever is very important as well.

If an access token or refresh token is compromised, the first thing you should do is go to the admin console and push a not-before revocation policy to all applications. This will enforce that any tokens issued prior to that date are now invalid. Pushing new not-before policy will also ensure that application will be forced to download new public keys from Keycloak, hence it is also useful for the case, when you think that realm signing key was compromised. More info in the [keys chapter](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/realms/keys.html#\_realm\_keys).

You can also disable specific applications, clients, and users if you feel that any one of those entities is completely compromised.
