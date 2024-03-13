# Keycloak Adapter Policy Enforcer

You can enforce authorization decisions for your applications if you are using Keycloak OIDC adapters.

When you enable policy enforcement for your Keycloak application, the corresponding adapter intercepts all requests to your application and enforces the authorization decisions obtained from the server.

Policy enforcement is strongly linked to your application’s paths and the [resources](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/resource/overview.html#\_resource\_overview) you created for a resource server using the Keycloak Administration Console. By default, when you create a resource server, Keycloak creates a [default configuration](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/resource-server/default-config.html#\_resource\_server\_default\_config) for your resource server so you can enable policy enforcement quickly.

The default configuration allows access for all resources in your application provided the authenticated user belongs to the same realm as the resource server being protected.

**Policy Enforcement Configuration**

To enable policy enforcement for your application, add the following property to your **keycloak.json** file:

keycloak.json

```json
{
  "policy-enforcer": {}
}
```

Or a little more verbose if you want to manually define the resources being protected:

```json
{
  "policy-enforcer": {
    "user-managed-access" : {},
    "enforcement-mode" : "ENFORCING"
    "paths": [
      {
        "path" : "/someUri/*",
        "methods" : [
          {
            "method": "GET",
            "scopes" : ["urn:app.com:scopes:view"]
          },
          {
            "method": "POST",
            "scopes" : ["urn:app.com:scopes:create"]
          }
        ]
      },
      {
        "name" : "Some Resource",
        "path" : "/usingPattern/{id}",
        "methods" : [
          {
            "method": "DELETE",
            "scopes" : ["urn:app.com:scopes:delete"]
          }
        ]
      },
      {
        "path" : "/exactMatch"
      },
      {
        "name" : "Admin Resources",
        "path" : "/usingWildCards/*"
      }
    ]
  }
}
```

Here is a description of each configuration option:

*   **policy-enforcer**

    Specifies the configuration options that define how policies are actually enforced and optionally the paths you want to protect. If not specified, the policy enforcer queries the server for all resources associated with the resource server being protected. In this case, you need to ensure the resources are properly configured with a [URI](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/resource/create.html#\_resource\_create\_uri) property that matches the paths you want to protect.

    *   **user-managed-access**

        Specifies that the adapter uses the UMA protocol. If specified, the adapter queries the server for permission tickets and return them to clients according to the UMA specification. If not specified, the adapter relies on the requesting party token (RPT) sent to the server to enforce permissions.
    *   **enforcement-mode**

        Specifies how policies are enforced.

        *   **ENFORCING**

            (default mode) Requests are denied by default even when there is no policy associated with a given resource.
        *   **PERMISSIVE**

            Requests are allowed even when there is no policy associated with a given resource.
        *   **DISABLED**

            Completely disables the evaluation of policies and allows access to any resource.
    *   **on-deny-redirect-to**

        Defines a URL where a client request is redirected when an "access denied" message is obtained from the server. By default, the adapter responds with a 403 HTTP status code.
    *   **paths**

        Specifies the paths to protect.

        *   **name**

            The name of a resource on the server that is to be associated with a given path. When used in conjunction with a **path**, the policy enforcer ignores the resource’s **URI** property and uses the path you provided instead.
        *   **path**

            (required) A URI relative to the application’s context path. If this option is specified, the policy enforcer queries the server for a resource with a **URI** with the same value. Currently a very basic logic for path matching is supported. Examples of valid paths are:

            * Wildcards: `/*`
            * Suffix: `/*.html`
            * Sub-paths: `/path/*`
            * Path parameters: /resource/{id}
            * Exact match: /resource
            * Patterns: /{version}/resource, /api/{version}/resource, /api/{version}/resource/\*
        * **methods** The HTTP methods (for example, GET, POST, PATCH) to protect and how they are associated with the scopes for a given resource in the server. +\[/'']
          *   **method**

              The name of the HTTP method.
          *   **scopes**

              An array of strings with the scopes associated with the method. When you associate scopes with a specific method, the client trying to access a protected resource (or path) must provide an RPT that grants permission to all scopes specified in the list. For example, if you define a method _POST_ with a scope _create_, the RPT must contain a permission granting access to the _create_ scope when performing a POST to the path.
        *   **enforcement-mode**

            Specifies how policies are enforced.

            *   **ENFORCING**

                (default mode) Requests are denied by default even when there is no policy associated with a given resource.
            *   **DISABLED**

                Disables the evaluation of policies for a path
