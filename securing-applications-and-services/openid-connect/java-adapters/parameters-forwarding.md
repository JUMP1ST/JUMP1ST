# Parameters Forwarding

The Keycloak initial authorization endpoint request has support for various parameters. Most of the parameters are described in [OIDC specification](http://openid.net/specs/openid-connect-core-1\_0.html#AuthorizationEndpoint). Some parameters are added automatically by the adapter based on the adapter configuration. However, there are also a few parameters that can be added on a per-invocation basis. When you open the secured application URI, the particular parameter will be forwarded to the Keycloak authorization endpoint.

For example, if you request an offline token, then you can open the secured application URI with the `scope` parameter like:

```
http://myappserver/mysecuredapp?scope=offline_access
```

and the parameter `scope=offline_access` will be automatically forwarded to the Keycloak authorization endpoint.

The supported parameters are:

* scope
* prompt
* max\_age
* login\_hint
* kc\_idp\_hint

Most of the parameters are described in the [OIDC specification](http://openid.net/specs/openid-connect-core-1\_0.html#AuthorizationEndpoint). The only exception is parameter `kc_idp_hint`, which is specific to Keycloak and contains the name of the identity provider to automatically use. For more information see the `Identity Brokering` section in [Server Administration](https://keycloak.gitbooks.io/documentation/content/server\_admin/index.html).
