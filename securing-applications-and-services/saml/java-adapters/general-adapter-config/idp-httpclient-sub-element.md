# IDP HttpClient sub element

The `HttpClient` optional sub element defines the properties of HTTP client used for automatic obtaining of certificates containing public keys for IDP signature verification via SAML descriptor of the IDP when [enabled](https://wjw465150.gitbooks.io/keycloak-documentation/content/securing\_apps/topics/saml/java/general-config/idp-keys\_subelement.html#\_sp-idp-keys-automatic).

```xml
<HttpClient connectionPoolSize="10"
            disableTrustManager="false"
            allowAnyHostname="false"
            clientKeystore="classpath:keystore.jks"
            clientKeystorePassword="pwd"
            truststore="classpath:truststore.jks"
            truststorePassword="pwd"
            proxyUrl="http://proxy/" />
```

connectionPoolSize

Adapters will make separate HTTP invocations to the Keycloak server to turn an access code into an access token. This config option defines how many connections to the Keycloak server should be pooled. This is _OPTIONAL_. The default value is `10`.

disableTrustManager

If the Keycloak server requires HTTPS and this config option is set to `true` you do not have to specify a truststore. This setting should only be used during development and **never** in production as it will disable verification of SSL certificates. This is _OPTIONAL_. The default value is `false`.

allowAnyHostname

If the Keycloak server requires HTTPS and this config option is set to `true` the Keycloak server’s certificate is validated via the truststore, but host name validation is not done. This setting should only be used during development and **never** in production as it will partly disable verification of SSL certificates. This seting may be useful in test environments. This is _OPTIONAL_. The default value is `false`.

truststore

The value is the file path to a keystore file. If you prefix the path with `classpath:`, then the truststore will be obtained from the deployment’s classpath instead. Used for outgoing HTTPS communications to the Keycloak server. Client making HTTPS requests need a way to verify the host of the server they are talking to. This is what the trustore does. The keystore contains one or more trusted host certificates or certificate authorities. You can create this truststore by extracting the public certificate of the Keycloak server’s SSL keystore. This is _REQUIRED_ unless `disableTrustManager` is `true`.

truststorePassword

Password for the truststore keystore. This is _REQUIRED_ if `truststore` is set and the truststore requires a password.

clientKeystore

This is the file path to a keystore file. This keystore contains client certificate for two-way SSL when the adapter makes HTTPS requests to the Keycloak server. This is _OPTIONAL_.

clientKeystorePassword

Password for the client keystore and for the client’s key. This is _REQUIRED_ if `clientKeystore` is set.

proxyUrl

URL to HTTP proxy to use for HTTP connections. This is _OPTIONAL_.

\
