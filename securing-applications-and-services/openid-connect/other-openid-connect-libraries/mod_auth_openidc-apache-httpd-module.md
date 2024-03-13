# mod\_auth\_openidc Apache HTTPD Module

The [mod\_auth\_openidc](https://github.com/pingidentity/mod\_auth\_openidc) is an Apache HTTP plugin for OpenID Connect. If your language/environment supports using Apache HTTPD as a proxy, then you can use _mod\_auth\_openidc_ to secure your web application with OpenID Connect. Configuration of this module is beyond the scope of this document. Please see the _mod\_auth\_openidc_ Github repo for more details on configuration.

To configure _mod\_auth\_openidc_ you’ll need

* The client\_id.
* The client\_secret.
* The redirect\_uri to your application.
* The Keycloak openid-configuration url
* _mod\_auth\_openidc_ specific Apache HTTPD module config.

An example configuration would look like the following.

```xml
LoadModule auth_openidc_module modules/mod_auth_openidc.so

ServerName ${HOSTIP}

<VirtualHost *:80>

    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html

    #this is required by mod_auth_openidc
    OIDCCryptoPassphrase a-random-secret-used-by-apache-oidc-and-balancer

    OIDCProviderMetadataURL ${KC_ADDR}/auth/realms/${KC_REALM}/.well-known/openid-configuration

    OIDCClientID ${CLIENT_ID}
    OIDCClientSecret ${CLIENT_SECRET}
    OIDCRedirectURI http://${HOSTIP}/${CLIENT_APP_NAME}/redirect_uri

    # maps the prefered_username claim to the REMOTE_USER environment variable
    OIDCRemoteUserClaim preferred_username

    <Location /${CLIENT_APP_NAME}/>
        AuthType openid-connect
        Require valid-user
    </Location>
</VirtualHost>
```

Further information on how to configure mod\_auth\_openidc can be found on the [mod\_auth\_openidc](https://github.com/pingidentity/mod\_auth\_openidc) project page.

\
