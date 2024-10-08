# Java Adapter Config

Each Java adapter supported by Keycloak can be configured by a simple JSON file. This is what one might look like:

```json
{
  "realm" : "demo",
  "resource" : "customer-portal",
  "realm-public-key" : "MIGfMA0GCSqGSIb3D...31LwIDAQAB",
  "auth-server-url" : "https://localhost:8443/auth",
  "ssl-required" : "external",
  "use-resource-role-mappings" : false,
  "enable-cors" : true,
  "cors-max-age" : 1000,
  "cors-allowed-methods" : "POST, PUT, DELETE, GET",
  "cors-exposed-headers" : "WWW-Authenticate, My-custom-exposed-Header",
  "bearer-only" : false,
  "enable-basic-auth" : false,
  "expose-token" : true,
   "credentials" : {
      "secret" : "234234-234234-234234"
   },

   "connection-pool-size" : 20,
   "disable-trust-manager": false,
   "allow-any-hostname" : false,
   "truststore" : "path/to/truststore.jks",
   "truststore-password" : "geheim",
   "client-keystore" : "path/to/client-keystore.jks",
   "client-keystore-password" : "geheim",
   "client-key-password" : "geheim",
   "token-minimum-time-to-live" : 10,
   "min-time-between-jwks-requests" : 10,
   "public-key-cache-ttl": 86400
}
```

You can use `${…​}` enclosure for system property replacement. For example `${jboss.server.config.dir}` would be replaced by `/path/to/Keycloak`. Replacement of environment variables is also supported via the `env` prefix, e.g. `${env.MY_ENVIRONMENT_VARIABLE}`.

The initial config file can be obtained from the the admin console. This can be done by opening the admin console, select `Clients` from the menu and clicking on the corresponding client. Once the page for the client is opened click on the `Installation` tab and select `Keycloak OIDC JSON`.

Here is a description of each configuration option:

realm

Name of the realm. This is _REQUIRED._

resource

The client-id of the application. Each application has a client-id that is used to identify the application. This is _REQUIRED._

realm-public-key

PEM format of the realm public key. You can obtain this from the administration console. This is _OPTIONAL_ and it’s not recommended to set it. If not set, the adapter will download this from Keycloak and it will always re-download it when needed (eg. Keycloak rotate it’s keys). However if realm-public-key is set, then adapter will never download new keys from Keycloak, so when Keycloak rotate it’s keys, adapter will break.

auth-server-url

The base URL of the Keycloak server. All other Keycloak pages and REST service endpoints are derived from this. It is usually of the form `https://host:port/auth`. This is _REQUIRED._

ssl-required

Ensures that all communication to and from the Keycloak server is over HTTPS. In production this should be set to `all`. This is _OPTIONAL_. The default value is _external_ meaning that HTTPS is required by default for external requests. Valid values are 'all', 'external' and 'none'.

use-resource-role-mappings

If set to true, the adapter will look inside the token for application level role mappings for the user. If false, it will look at the realm level for user role mappings. This is _OPTIONAL_. The default value is _false_.

public-client

If set to true, the adapter will not send credentials for the client to Keycloak. This is _OPTIONAL_. The default value is _false_.

enable-cors

This enables CORS support. It will handle CORS preflight requests. It will also look into the access token to determine valid origins. This is _OPTIONAL_. The default value is _false_.

cors-max-age

If CORS is enabled, this sets the value of the `Access-Control-Max-Age` header. This is _OPTIONAL_. If not set, this header is not returned in CORS responses.

cors-allowed-methods

If CORS is enabled, this sets the value of the `Access-Control-Allow-Methods` header. This should be a comma-separated string. This is _OPTIONAL_. If not set, this header is not returned in CORS responses.

cors-allowed-headers

If CORS is enabled, this sets the value of the `Access-Control-Allow-Headers` header. This should be a comma-separated string. This is _OPTIONAL_. If not set, this header is not returned in CORS responses.

cors-exposed-headers

If CORS is enabled, this sets the value of the `Access-Control-Expose-Headers` header. This should be a comma-separated string. This is _OPTIONAL_. If not set, this header is not returned in CORS responses.

bearer-only

This should be set to _true_ for services. If enabled the adapter will not attempt to authenticate users, but only verify bearer tokens. This is _OPTIONAL_. The default value is _false_.

autodetect-bearer-only

This should be set to _true_ if your application serves both a web application and web services (e.g. SOAP or REST). It allows you to redirect unauthenticated users of the web application to the Keycloak login page, but send an HTTP `401` status code to unauthenticated SOAP or REST clients instead as they would not understand a redirect to the login page. Keycloak auto-detects SOAP or REST clients based on typical headers like `X-Requested-With`, `SOAPAction` or `Accept`. The default value is _false_.

enable-basic-auth

This tells the adapter to also support basic authentication. If this option is enabled, then _secret_ must also be provided. This is _OPTIONAL_. The default value is _false_.

expose-token

If `true`, an authenticated browser client (via a Javascript HTTP invocation) can obtain the signed access token via the URL `root/k_query_bearer_token`. This is _OPTIONAL_. The default value is _false_.

credentials

Specify the credentials of the application. This is an object notation where the key is the credential type and the value is the value of the credential type. Currently password and jwt is supported. This is _REQUIRED_ only for clients with 'Confidential' access type.

connection-pool-size

Adapters will make separate HTTP invocations to the Keycloak server to turn an access code into an access token. This config option defines how many connections to the Keycloak server should be pooled. This is _OPTIONAL_. The default value is `20`.

disable-trust-manager

If the Keycloak server requires HTTPS and this config option is set to `true` you do not have to specify a truststore. This setting should only be used during development and **never** in production as it will disable verification of SSL certificates. This is _OPTIONAL_. The default value is `false`.

allow-any-hostname

If the Keycloak server requires HTTPS and this config option is set to `true` the Keycloak server’s certificate is validated via the truststore, but host name validation is not done. This setting should only be used during development and **never** in production as it will disable verification of SSL certificates. This seting may be useful in test environments This is _OPTIONAL_. The default value is `false`.

proxy-url

The URL for the HTTP proxy if one is used.

truststore

The value is the file path to a keystore file. If you prefix the path with `classpath:`, then the truststore will be obtained from the deployment’s classpath instead. Used for outgoing HTTPS communications to the Keycloak server. Client making HTTPS requests need a way to verify the host of the server they are talking to. This is what the trustore does. The keystore contains one or more trusted host certificates or certificate authorities. You can create this truststore by extracting the public certificate of the Keycloak server’s SSL keystore. This is _REQUIRED_ unless `ssl-required` is `none` or `disable-trust-manager` is `true`.

truststore-password

Password for the truststore keystore. This is _REQUIRED_ if `truststore` is set and the truststore requires a password.

client-keystore

This is the file path to a keystore file. This keystore contains client certificate for two-way SSL when the adapter makes HTTPS requests to the Keycloak server. This is _OPTIONAL_.

client-keystore-password

Password for the client keystore. This is _REQUIRED_ if `client-keystore` is set.

client-key-password

Password for the client’s key. This is _REQUIRED_ if `client-keystore` is set.

always-refresh-token

If _true_, the adapter will refresh token in every request.

register-node-at-startup

If _true_, then adapter will send registration request to Keycloak. It’s _false_ by default and useful only when application is clustered. See [Application Clustering](https://wjw465150.gitbooks.io/keycloak-documentation/content/securing\_apps/topics/oidc/oid/java/application-clustering.html#\_applicationclustering) for details

register-node-period

Period for re-registration adapter to Keycloak. Useful when application is clustered. See [Application Clustering](https://wjw465150.gitbooks.io/keycloak-documentation/content/securing\_apps/topics/oidc/oid/java/application-clustering.html#\_applicationclustering) for details

token-store

Possible values are _session_ and _cookie_. Default is _session_, which means that adapter stores account info in HTTP Session. Alternative _cookie_ means storage of info in cookie. See [Application Clustering](https://wjw465150.gitbooks.io/keycloak-documentation/content/securing\_apps/topics/oidc/oid/java/application-clustering.html#\_applicationclustering) for details

principal-attribute

OpenID Connection ID Token attribute to populate the UserPrincipal name with. If token attribute is null, defaults to `sub`. Possible values are `sub`, `preferred_username`, `email`, `name`, `nickname`, `given_name`, `family_name`.

turn-off-change-session-id-on-login

The session id is changed by default on a successful login on some platforms to plug a security attack vector. Change this to true if you want to turn this off This is _OPTIONAL_. The default value is _false_.

token-minimum-time-to-live

Amount of time, in seconds, to preemptively refresh an active access token with the Keycloak server before it expires. This is especially useful when the access token is sent to another REST client where it could expire before being evaluated. This value should never exceed the realm’s access token lifespan. This is _OPTIONAL_. The default value is `0` seconds, so adapter will refresh access token just if it’s expired.

min-time-between-jwks-requests

Amount of time, in seconds, specifying minimum interval between two requests to Keycloak to retrieve new public keys. It is 10 seconds by default. Adapter will always try to download new public key when it recognize token with unknown `kid` . However it won’t try it more than once per 10 seconds (by default). This is to avoid DoS when attacker sends lots of tokens with bad `kid` forcing adapter to send lots of requests to Keycloak.

public-key-cache-ttl

Amount of time, in seconds, specifying maximum interval between two requests to Keycloak to retrieve new public keys. It is 86400 seconds (1 day) by default. Adapter will always try to download new public key when it recognize token with unknown `kid` . If it recognize token with known `kid`, it will just use the public key downloaded previously. However at least once per this configured interval (1 day by default) will be new public key always downloaded even if the `kid` of token is already known.

ignore-oauth-query-parameter

Defaults to `false`, if set to `true` will turn off processing of the `access_token` query parameter for bearer token processing. Users will not be able to authenticate if they only pass in an `access_token`
