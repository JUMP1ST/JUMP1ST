# Client Links

For scenarios where one wants to link from one client to another, Keycloak provides a special redirect endpoint: `/realms/realm_name/clients/{client-id}/redirect`.

If a client accesses this endpoint via an `HTTP GET` request, Keycloak returns the configured base URL for the provided Client and Realm in the form of an `HTTP 307` (Temporary Redirect) via the response’s `Location` header.

Thus, a client only needs to know the Realm name and the Client ID in order to link to them. This indirection helps avoid hard-coding client base URLs.

As an example, given the realm `master` and the client-id `account`:

```
http://host:port/auth/realms/master/clients/account/redirect
```

Would temporarily redirect to: http://host:port/auth/realms/master/account
