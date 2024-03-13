# Confidential Client Credentials

If you’ve set the client’s [access type](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/clients/client-oidc.html#\_access-type) to `confidential` in the client’s `Settings` tab, a new `Credentials` tab will show up. As part of dealing with this type of client you have to configure the client’s credentials.

Credentials Tab

![client-credentials.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/client-credentials.png)

The `Client Authenticator` list box specifies the type of credential you are going to use for your confidential client. It defaults to client ID and secret. The secret is automatically generated for you and the `Regenerate Secret` button allows you to recreate this secret if you want or need to.

Alternatively, you can opt to use a signed Json Web Token (JWT) instead of a secret.

Signed JWT

![client-credentials-jwt.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/client-credentials-jwt.png)

When choosing this credential type you will have to also generate a private key and certificate for the client. The private key will be used to sign the JWT, while the certificate is used by the server to verify the signature. Click on the `Generate new keys and certificate` button to start this process.

Generate Keys

![generate-client-keys.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/generate-client-keys.png)

When you generate these keys, Keycloak will store the certificate, and you’ll need to download the private key and certificate for your client to use. Pick the archive format you want and specify the password for the private key and store.

You can also opt to generate these via an external tool and just import the client’s certificate.

Import Certificate

![import-client-cert.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/import-client-cert.png)

There are multiple formats you can import from, just choose the archive format you have the certificate stored in, select the file, and click the `Import` button.

Finally note that you don’t even need to import certificate if you choose to `Use JWKS URL` . In that case, you can provide the URL where client publishes it’s public key in [JWK](https://self-issued.info/docs/draft-ietf-jose-json-web-key.html) format. This is flexible because when client changes it’s keys, Keycloak will automatically download them without need to re-import anything on Keycloak side.

If you use client secured by Keycloak adapter, you can configure the JWKS URL like [https://myhost.com/myapp/k\_jwks](https://myhost.com/myapp/k\_jwks) assuming that [https://myhost.com/myapp](https://myhost.com/myapp) is the root URL of your client application. See [Server Development](https://keycloak.gitbooks.io/documentation/content/server\_development/index.html) for additional details.

| Warning | For the performance purposes, Keycloak caches the public keys of the OIDC clients. If you think that private key of your client was compromised, it is obviously good to update your keys, but it’s also good to clear the keys cache. See [Clearing the cache](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/clients/realms/cache.html#\_clear-cache) section for more details. |
| ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
