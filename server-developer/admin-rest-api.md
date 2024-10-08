# Admin REST API

Keycloak comes with a fully functional Admin REST API with all features provided by the Admin Console.

To invoke the API you need to obtain an access token with the appropriate permissions. The required permissions are described in [Server Administration](https://keycloak.gitbooks.io/documentation/content/server\_admin/index.html).

A token can be obtained by enabling authenticating to your application with Keycloak; see the [Securing Applications and Services Guide](https://keycloak.gitbooks.io/documentation/content/securing\_apps/index.html). You can also use direct access grant to obtain an access token.

For complete documentation see [API Documentation](http://www.keycloak.org/documentation.html).

#### Example using CURL <a href="#example_using_curl" id="example_using_curl"></a>

Obtain access token for user in the realm `master` with username `admin` and password `password`:

```bash
curl \
  -d "client_id=admin-cli" \
  -d "username=admin" \
  -d "password=password" \
  -d "grant_type=password" \
  "http://localhost:8080/auth/realms/master/protocol/openid-connect/token"
```

| Note | By default this token expires in 1 minute |
| ---- | ----------------------------------------- |

The result will be a JSON document. To invoke the API you need to extract the value of the `access_token` property. You can then invoke the API by including the value in the `Authorization` header of requests to the API.

The following example shows how to get the details of the master realm:

```bash
curl \
  -H "Authorization: bearer eyJhbGciOiJSUz..." \
  "http://localhost:8080/auth/admin/realms/master"
```

#### Example using Java <a href="#example_using_java" id="example_using_java"></a>

There’s a Java client library for the Admin REST API that makes it easy to use from Java. To use it from your application add a dependency on the `keycloak-admin-client` library.

The following example shows how to use the Java client library to get the details of the master realm:

```java
import org.keycloak.admin.client.Keycloak;
import org.keycloak.representations.idm.RealmRepresentation;
...

Keycloak keycloak = Keycloak.getInstance(
    "http://localhost:8080/auth",
    "master",
    "admin",
    "password",
    "admin-cli");
RealmRepresentation realm = keycloak.realm("master").toRepresentation();
```

Complete Javadoc for the admin client is available at [API Documentation](http://www.keycloak.org/documentation.html).
