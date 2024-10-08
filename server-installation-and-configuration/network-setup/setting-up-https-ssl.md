# Setting up HTTPS/SSL

| Warning | Keycloak is not set up by default to handle SSL/HTTPS. It is highly recommended that you either enable SSL on the Keycloak server itself or on a reverse proxy in front of the Keycloak server. |
| ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

This default behavior is defined by the SSL/HTTPS mode of each Keycloak realm. This is discussed in more detail in the [Server Administration](https://keycloak.gitbooks.io/documentation/content/server\_admin/index.html), but let’s give some context and a brief overview of these modes.

external requests

Keycloak can run out of the box without SSL so long as you stick to private IP addresses like `localhost`, `127.0.0.1`, `10.0.x.x`, `192.168.x.x`, and `172..16.x.x`. If you don’t have SSL/HTTPS configured on the server or you try to access Keycloak over HTTP from a non-private IP adress you will get an error.

none

Keycloak does not require SSL. This should really only be used in development when you are playing around with things.

all requests

Keycloak requires SSL for all IP addresses.

The SSL mode for each realm can be configured in the Keycloak admin console.

**Enabling SSL/HTTPS for the Keycloak Server**

If you are not using a reverse proxy or load balancer to handle HTTPS traffic for you, you’ll need to enable HTTPS for the Keycloak server. This involves

1. Obtaining or generating a keystore that contains the private key and certificate for SSL/HTTP traffic
2. Configuring the Keycloak server to use this keypair and certificate.

**Creating the Certificate and Java Keystore**

In order to allow HTTPS connections, you need to obtain a self signed or third-party signed certificate and import it into a Java keystore before you can enable HTTPS in the web container you are deploying the Keycloak Server to.

**Self Signed Certificate**

In development, you will probably not have a third party signed certificate available to test a Keycloak deployment so you’ll need to generate a self-signed one using the `keytool` utility that comes with the Java JDK.

```
$ keytool -genkey -alias localhost -keyalg RSA -keystore keycloak.jks -validity 10950
    Enter keystore password: secret
    Re-enter new password: secret
    What is your first and last name?
    [Unknown]:  localhost
    What is the name of your organizational unit?
    [Unknown]:  Keycloak
    What is the name of your organization?
    [Unknown]:  Red Hat
    What is the name of your City or Locality?
    [Unknown]:  Westford
    What is the name of your State or Province?
    [Unknown]:  MA
    What is the two-letter country code for this unit?
    [Unknown]:  US
    Is CN=localhost, OU=Keycloak, O=Test, L=Westford, ST=MA, C=US correct?
    [no]:  yes
```

You should answer `What is your first and last name ?` question with the DNS name of the machine you’re installing the server on. For testing purposes, `localhost` should be used. After executing this command, the `keycloak.jks` file will be generated in the same directory as you executed the `keytool` command in.

If you want a third-party signed certificate, but don’t have one, you can obtain one for free at [cacert.org](http://cacert.org/). You’ll have to do a little set up first before doing this though.

The first thing to do is generate a Certificate Request:

```
$ keytool -certreq -alias yourdomain -keystore keycloak.jks > keycloak.careq
```

Where `yourdomain` is a DNS name for which this certificate is generated for. Keytool generates the request:

```
-----BEGIN NEW CERTIFICATE REQUEST-----
MIIC2jCCAcICAQAwZTELMAkGA1UEBhMCVVMxCzAJBgNVBAgTAk1BMREwDwYDVQQHEwhXZXN0Zm9y
ZDEQMA4GA1UEChMHUmVkIEhhdDEQMA4GA1UECxMHUmVkIEhhdDESMBAGA1UEAxMJbG9jYWxob3N0
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAr7kck2TaavlEOGbcpi9c0rncY4HhdzmY
Ax2nZfq1eZEaIPqI5aTxwQZzzLDK9qbeAd8Ji79HzSqnRDxNYaZu7mAYhFKHgixsolE3o5Yfzbw1
29RvyeUVe+WZxv5oo9wolVVpdSINIMEL2LaFhtX/c1dqiqYVpfnvFshZQaIg2nL8juzZcBjj4as
H98gIS7khql/dkZKsw9NLvyxgJvp7PaXurX29fNf3ihG+oFrL22oFyV54BWWxXCKU/GPn61EGZGw
Ft2qSIGLdctpMD1aJR2bcnlhEjZKDksjQZoQ5YMXaAGkcYkG6QkgrocDE2YXDbi7GIdf9MegVJ35
2DQMpwIDAQABoDAwLgYJKoZIhvcNAQkOMSEwHzAdBgNVHQ4EFgQUQwlZJBA+fjiDdiVzaO9vrE/i
n2swDQYJKoZIhvcNAQELBQADggEBAC5FRvMkhal3q86tHPBYWBuTtmcSjs4qUm6V6f63frhveWHf
PzRrI1xH272XUIeBk0gtzWo0nNZnf0mMCtUBbHhhDcG82xolikfqibZijoQZCiGiedVjHJFtniDQ
9bMDUOXEMQ7gHZg5q6mJfNG9MbMpQaUVEEFvfGEQQxbiFK7hRWU8S23/d80e8nExgQxdJWJ6vd0X
MzzFK6j4Dj55bJVuM7GFmfdNC52pNOD5vYe47Aqh8oajHX9XTycVtPXl45rrWAH33ftbrS8SrZ2S
vqIFQeuLL3BaHwpl3t7j2lMWcK1p80laAxEASib/fAwrRHpLHBXRcq6uALUOZl4Alt8=
-----END NEW CERTIFICATE REQUEST-----
```

Send this ca request to your CA. The CA will issue you a signed certificate and send it to you. Before you import your new cert, you must obtain and import the root certificate of the CA. You can download the cert from CA (ie.: root.crt) and import as follows:

```
$ keytool -import -keystore keycloak.jks -file root.crt -alias root
```

Last step is to import your new CA generated certificate to your keystore:

```
$ keytool -import -alias yourdomain -keystore keycloak.jks -file your-certificate.cer
```

**Configure Keycloak to Use the Keystore**

Now that you have a Java keystore with the appropriate certificates, you need to configure your Keycloak installation to use it. First step is to move the keystore file to the _configuration/_ directory of your deployment and to edit the _standalone.xml_, _standalone-ha.xml_ or _domain.xml_ file to use the keystore and enable HTTPS. (See [operating mode](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/operating-mode.html#\_operating-mode)).

In the standalone or domain configuration file, search for the `security-realms` element and add:

```xml
<security-realm name="UndertowRealm">
    <server-identities>
        <ssl>
            <keystore path="keycloak.jks" relative-to="jboss.server.config.dir" keystore-password="secret" />
        </ssl>
    </server-identities>
</security-realm>
```

Find the element `server name="default-server"` (it’s a child element of `subsystem xmlns="urn:jboss:domain:undertow:3.0"`) and add:

```xml
<subsystem xmlns="urn:jboss:domain:undertow:3.0">
   <buffer-cache name="default"/>
   <server name="default-server">
      <https-listener name="https" socket-binding="https" security-realm="UndertowRealm"/>
   ...
</subsystem>
```
