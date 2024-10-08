# Authorization Client Java API

Depending on your requirements, a resource server should be able to manage resources remotely or even check for permissions programmatically. If you are using Java, you can access the Keycloak Authorization Services using the Authorization Client API.

It is targeted for resource servers that want to access the different APIs provided by the server such as the Protection, Authorization and Entitlement APIs.

**Maven Dependency**

```xml
<dependencies>
    <dependency>
        <groupId>org.keycloak</groupId>
        <artifactId>keycloak-authz-client</artifactId>
        <version>${KEYCLOAK_VERSION}</version>
    </dependency>
</dependencies>
```

**Configuration**

The client configuration is defined in a `keycloak.json` file as follows:

```json
{
  "realm": "hello-world-authz",
  "auth-server-url" : "http://localhost:8080/auth",
  "resource" : "hello-world-authz-service",
  "credentials": {
    "secret": "secret"
  }
}
```

*   **realm** (required)

    The name of the realm.
*   **auth-server-url** (required)

    The base URL of the Keycloak server. All other Keycloak pages and REST service endpoints are derived from this. It is usually in the form https://host:port/auth.
*   **resource** (required)

    The client-id of the application. Each application has a client-id that is used to identify the application.
* **credentials** (required) Specifies the credentials of the application. This is an object notation where the key is the credential type and the value is the value of the credential type.

The configuration file is usually located in your application’s classpath, the default location from where the client is going to try to find a `keycloak.json` file.

**Creating the Authorization Client**

Considering you have a `keycloak.json` file in your classpath, you can create a new `AuthzClient` instance as follows:

```java
    // create a new instance based on the configuration defined in a keycloak.json located in your classpath
    AuthzClient authzClient = AuthzClient.create();
```

**Obtaining User Entitlements**

Here is an example illustrating how to obtain user entitlements:

```java
// create a new instance based on the configuration defined in keycloak-authz.json
AuthzClient authzClient = AuthzClient.create();

// obtain an Entitlement API Token to get access to the Entitlement API.
// this token is an access token issued to a client on behalf of an user
// with a uma_authorization scope
String eat = getEntitlementAPIToken(authzClient);

// send the entitlement request to the server to
// obtain an RPT with all permissions granted to the user
EntitlementResponse response = authzClient.entitlement(eat).getAll("hello-world-authz-service");
String rpt = response.getRpt();

// now you can use the RPT to access protected resources on the resource server
```

Here is an example illustrating how to obtain user entitlements for a set of one or more resources:

```java
// create a new instance based on the configuration defined in keycloak-authz.json
AuthzClient authzClient = AuthzClient.create();

// obtain an Entitlement API Token to get access to the Entitlement API.
// this token is an access token issued to a client on behalf of an user
// with a uma_authorization scope
String eat = getEntitlementAPIToken(authzClient);

// create an entitlement request
EntitlementRequest request = new EntitlementRequest();
PermissionRequest permission = new PermissionRequest();

permission.setResourceSetName("Hello World Resource");

request.addPermission(permission);

// send the entitlement request to the server to obtain an RPT
// with all permissions granted to the user
EntitlementResponse response = authzClient.entitlement(eat).get("hello-world-authz-service", request);
String rpt = response.getRpt();

// now you can use the RPT to access protected resources on the resource server
```

**Creating a Resource Using the Protection API**

```java
// create a new instance based on the configuration defined in keycloak-authz.json
AuthzClient authzClient = AuthzClient.create();

// create a new resource representation with the information we want
ResourceRepresentation newResource = new ResourceRepresentation();

newResource.setName("New Resource");
newResource.setType("urn:hello-world-authz:resources:example");

newResource.addScope(new ScopeRepresentation("urn:hello-world-authz:scopes:view"));

ProtectedResource resourceClient = authzClient.protection().resource();
Set<String> existingResource = resourceClient.findByFilter("name=" + newResource.getName());

if (!existingResource.isEmpty()) {
    resourceClient.delete(existingResource.iterator().next());
}

// create the resource on the server
RegistrationResponse response = resourceClient.create(newResource);
String resourceId = response.getId();

// query the resource using its newly generated id
ResourceRepresentation resource = resourceClient.findById(resourceId).getResourceDescription();
```

**Introspecting a RPT**

```java
    AuthzClient authzClient = AuthzClient.create();
    String rpt = getRequestingPartyToken(authzClient);
    TokenIntrospectionResponse requestingPartyToken = authzClient.protection().introspectRequestingPartyToken(rpt);

    if (requestingPartyToken.getActive()) {
        for (Permission granted : requestingPartyToken.getPermissions()) {
            // iterate over the granted permissions
        }
    }
```

\
