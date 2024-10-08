# Client Roles

Client roles are basically a namespace dedicated to a client. Each client gets its own namespace. Client roles are managed under the `Roles` tab under each individual client. You interact with this UI the same way you do for realm-level roles.

If the client has to explicitly request another client’s role, the role has to be prefixed with the client ID when performing a request using the scope parameter. For example, if the client ID is `account` and the role is `admin`, the scope parameter is:

```
`scope=account/admin`
```

As noted in the realm roles section, multiple roles are separated by spaces.
