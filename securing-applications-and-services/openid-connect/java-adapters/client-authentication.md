# Client Authentication

When a confidential OIDC client needs to send a backchannel request (for example, to exchange code for the token, or to refresh the token) it needs to authenticate against the Keycloak server. By default, there are two ways to authenticate the client: client ID and client secret, or client authentication with signed JWT.

**Client ID and Client Secret**

This is the traditional method described in the OAuth2 specification. The client has a secret, which needs to be known to both the adapter (application) and the Keycloak server. You can generate the secret for a particular client in the Keycloak administration console, and then paste this secret into the `keycloak.json` file on the application side:

```
"credentials": {
    "secret": "19666a4f-32dd-4049-b082-684c74115f28"
}
```

**Client Authentication with Signed JWT**

This is based on the [RFC7523](https://tools.ietf.org/html/rfc7523) specification. It works this way:

* The client must have the private key and certificate. For Keycloak this is available through the traditional `keystore` file, which is either available on the client application’s classpath or somewhere on the file system.
* Once the client application is started, it allows to download its public key in [JWKS](https://self-issued.info/docs/draft-ietf-jose-json-web-key.html) format using a URL such as http://myhost.com/myapp/k\_jwks, assuming that http://myhost.com/myapp is the base URL of your client application. This URL can be used by Keycloak (see below).
* During authentication, the client generates a JWT token and signs it with its private key and sends it to Keycloak in the particular backchannel request (for example, code-to-token request) in the `client_assertion` parameter.
* Keycloak must have the public key or certificate of the client so that it can verify the signature on JWT. In Keycloak you need to configure client credentials for your client. First you need to choose `Signed JWT` as the method of authenticating your client in the tab `Credentials` in administration console. Then you can choose to either:
  * Configure the JWKS URL where Keycloak can download the client’s public keys. This can be a URL such as http://myhost.com/myapp/k\_jwks (see details above). This option is the most flexible, since the client can rotate its keys anytime and Keycloak then always downloads new keys when needed without needing to change the configuration. More accurately, Keycloak downloads new keys when it sees the token signed by an unknown `kid` (Key ID).
  * Upload the client’s public key or certificate, either in PEM format, in JWK format, or from the keystore. With this option, the public key is hardcoded and must be changed when the client generates a new key pair. You can even generate your own keystore from the Keycloak admininstration console if you don’t have your own available. For more details on how to set up the Keycloak administration console see [Server Administration](https://keycloak.gitbooks.io/documentation/content/server\_admin/index.html).

For set up on the adapter side you need to have something like this in your `keycloak.json` file:

```
"credentials": {
  "jwt": {
    "client-keystore-file": "classpath:keystore-client.jks",
    "client-keystore-type": "JKS",
    "client-keystore-password": "storepass",
    "client-key-password": "keypass",
    "client-key-alias": "clientkey",
    "token-expiration": 10
  }
}
```

With this configuration, the keystore file `keystore-client.jks` must be available on classpath in your WAR. If you do not use the prefix `classpath:` you can point to any file on the file system where the client application is running.

For inspiration, you can take a look at the examples distribution into the main demo example into the `product-portal` application.

**Add Your Own Client Authentication Method**

You can add your own client authentication method as well. You will need to implement both client-side and server-side providers. For more details see the `Authentication SPI` section in [Server Development](https://keycloak.gitbooks.io/documentation/content/server\_development/index.html).
