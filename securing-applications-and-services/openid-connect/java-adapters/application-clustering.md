# Application Clustering

This chapter is related to supporting clustered applications deployed to JBoss EAP, WildFly and JBoss AS.

There are a few options available depending on whether your application is:

* Stateless or stateful
* Distributable (replicated http session) or non-distributable
* Relying on sticky sessions provided by load balancer
* Hosted on same domain as Keycloak

Dealing with clustering is not quite as simple as for a regular application. Mainly due to the fact that both the browser and the server-side application sends requests to Keycloak, so it’s not as simple as enabling sticky sessions on your load balancer.

**Stateless token store**

By default, the web application secured by Keycloak uses the HTTP session to store security context. This means that you either have to enable sticky sessions or replicate the HTTP session.

As an alternative to storing the security context in the HTTP session the adapter can be configured to store this in a cookie instead. This is useful if you want to make your application stateless or if you don’t want to store the security context in the HTTP session.

To use the cookie store for saving the security context, edit your applications `WEB-INF/keycloak.json` and add:

```json
"token-store": "cookie"
```

| Note | The default value for `token-store` is `session`, which stores the security context in the HTTP session. |
| ---- | -------------------------------------------------------------------------------------------------------- |

One limitation of using the cookie store is that the whole security context is passed in the cookie for every HTTP request. This may impact performance.

Another small limitation is limited support for Single-Sign Out. It works without issues if you init servlet logout (HttpServletRequest.logout) from the application itself as the adapter will delete the KEYCLOAK\_ADAPTER\_STATE cookie. However, back-channel logout initialized from a different application isn’t propagated by Keycloak to applications using cookie store. Hence it’s recommended to use a short value for the access token timeout (for example 1 minute).

**Relative URI optimization**

In deployment scenarios where Keycloak and the application is hosted on the same domain (through a reverse proxy or load balancer) it can be convenient to use relative URI options in your client configuration.

With relative URIs the URI is resolved as relative to the URL of the URL used to access Keycloak.

For example if the URL to your application is `https://acme.org/myapp` and the URL to Keycloak is `https://acme.org/auth`, then you can use the redirect-uri `/myapp` instead of `https://acme.org/myapp`.

**Admin URL configuration**

Admin URL for a particular client can be configured in the Keycloak Administration Console. It’s used by the Keycloak server to send backend requests to the application for various tasks, like logout users or push revocation policies.

For example the way backchannel logout works is:

1. User sends logout request from one application
2. The application sends logout request to Keycloak
3. The Keycloak server invalidates the user session
4. The Keycloak server then sends a backchannel request to application with an admin url that are associated with the session
5. When an application receives the logout request it invalidates the corresponding HTTP session

If admin URL contains `${application.session.host}` it will be replaced with the URL to the node associated with the HTTP session.

**Registration of application nodes**

The previous section describes how Keycloak can send logout request to node associated with a specific HTTP session. However, in some cases admin may want to propagate admin tasks to all registered cluster nodes, not just one of them. For example to push a new not before policy to the application or to logout all users from the application.

In this case Keycloak needs to be aware of all application cluster nodes, so it can send the event to all of them. To achieve this, we support auto-discovery mechanism:

1. When a new application node joins the cluster, it sends a registration request to the Keycloak server
2. The request may be re-sent to Keycloak in configured periodic intervals
3. If the Keycloak server doesn’t receive a re-registration request within a specified timeout then it automatically unregisters the specific node
4. The node is also unregistered in Keycloak when it sends an unregistration request, which is usually during node shutdown or application undeployment. This may not work properly for forced shutdown when undeployment listeners are not invoked, which results in the need for automatic unregistration

Sending startup registrations and periodic re-registration is disabled by default as it’s only required for some clustered applications.

To enable the feature edit the `WEB-INF/keycloak.json` file for your application and add:

```
"register-node-at-startup": true,
"register-node-period": 600,
```

This means the adapter will send the registration request on startup and re-register every 10 minutes.

In the Keycloak Administration Console you can specify the maximum node re-registration timeout (should be larger than _register-node-period_ from the adapter configuration). You can also manually add and remove cluster nodes in through the Adminstration Console, which is useful if you don’t want to rely on the automatic registration feature or if you want to remove stale application nodes in the event your not using the automatic unregistration feature.

**Refresh token in each request**

By default the application adapter will only refresh the access token when it’s expired. However, you can also configure the adapter to refresh the token on every request. This may have a performance impact as your application will send more requests to the Keycloak server.

To enable the feature edit the `WEB-INF/keycloak.json` file for your application and add:

```
"always-refresh-token": true
```

| Note | This may have a significant impact on performance. Only enable this feature if you can’t rely on backchannel messages to propagate logout and not before policies. Another thing to consider is that by default access tokens has a short expiration so even if logout is not propagated the token will expire within minutes of the logout. |
| ---- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

\
