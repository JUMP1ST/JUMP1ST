# Docker Registry v2 Authentication

| Note | Docker authentication is disabled by default. To enable see [Profiles](https://keycloak.gitbooks.io/documentation/content/server\_installation/topics/profiles.html). |
| ---- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

[Docker Registry V2 Authentciation](https://docs.docker.com/registry/spec/auth/) is an OIDC-Like protocol used to authenticate users against a Docker registry. Keycloak’s implementation of this protocol allows for a Keycloak authentication server to be used by a Docker client to authenticate against a registry. While this protocol uses fairly standard token and signature mechanisms, it has a few wrinkles that prevent it from being treated as a true OIDC implementation. The largest deviations include a very specific JSON format for requests and responses as well as the ability to understand how to map repository names and permissions to the OAuth scope mechanism.

**Docker Auth Flow**

The [Docker API documentation](https://docs.docker.com/registry/spec/auth/token/) best describes and illustrates this process, however a brief summary will be given below from the perspective of they Keycloak authentication server.

| Note | This flow assumes that a `docker login` command has already been performed |
| ---- | -------------------------------------------------------------------------- |

* The flow begins when the Docker client requests a resource from the Docker registry. If the resource is protected and no auth token is present in the request, the Docker registry server will respond to the client with a 401 + some information on required permissions and where to find the authorization server.
* The Docker client will construct an authentication request based on the 401 response from the Docker registry. The client will then use the locally cached credentials (from a previously run `docker login` command) as part of a [HTTP Basic Authentication](https://tools.ietf.org/html/rfc2617) request to the Keycloak authentication server.
* The Keycloak authentication server will attempt to authenticate the user and return a JSON body containing an OAuth-style Bearer token.
* The Docker client will get the bearer token from the JSON response and use it in the Authorization header to request the protected resource.
* When the Docker registry recieves the new request for the protected resource with the token from the Keycloak server, the registry validates the token and grants access to the requested resource (if appropriate).

**Keycloak Docker Registry v2 Authentication Server URI Endpoints**

Keycloak really only has one endpoint for all Docker auth v2 requests.

`http(s)://authserver.host/auth/realms/{realm-name}/protocol/docker-v2`
