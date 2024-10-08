# X.509 Client Certificate User Authentication

Keycloak supports login with a X.509 client certificate if the server is configured for mutual SSL authentication.

A typical workflow is as follows:

* A client sends an authentication request over SSL/TLS channel
* During SSL/TLS handshake, the server and the client exchange their x.509/v3 certificates
* The container (wildfly) validates the certificate PKIX path and the certificate expiration
* The x.509 client certificate authenticator validates the client certificate as follows:
  * Optionally checks the certificate revocation status using CRL and/or CRL Distribution Points
  * Optionally checks the Certificate revocation status using OCSP (Online Certificate Status Protocol)
  * Optinally validates whether the key usage in the certificate matches the expected key usage
  * Optionally validates whether the extended key usage in the certificate matches the expected extended key usage
* If any of the above checks fails, the x.509 authentication fails
* Otherwise, the authenticator extracts the certificate identity and maps it to an existing user
* Once the certificate is mapped to an existing user, the behavior diverges depending on the authentication flow:
  * In the Browser Flow, the server prompts the user to confirm identity or to ignore it and instead sign in with username/password
  * In the case of the Direct Grant Flow, the server signs in the user

#### Features <a href="#features" id="features"></a>

**Supported Certificate Identity Sources**

* Match SubjectDN using regular expression
* X500 Subject’s e-mail attribute
* X500 Subject’s Common Name attribute
* Match IssuerDN using regular expression
* X500 Issuer’s e-mail attribute
* X500 Issuer’s Common Name attribute
* Certificate Serial Number

**Regular Expressions**

The certificate identity can be extracted from either Subject DN or Issuer DN using a regular expression as a filter. For example, the regular expression below will match the e-mail attribute:

```
emailAddress=(.*?)(?:,|$)
```

The regular expression filtering is applicable only if the `Identity Source` is set to either `Match SubjectDN using regular expression` or `Match IssuerDN using regular expression`.

**Mapping certificate identity to an existing user**

The certificate identity mapping can be configured to map the extracted user identity to an existing user’s username or e-mail or to a custom attribute which value matches the certificate identity. For example, setting the `Identity source` to _Subject’s e-mail_ and `User mapping method` to _Username or email_ will have the X.509 client certificate authenticator use the e-mail attribute in the certificate’s Subject DN as a search criteria to look up an existing user by username or by e-mail.

**Other Features: Extended Certificate Validation**

* Revocation status checking using CRL
* Revocation status checking using CRL/Distribution Point
* Revocation status checking using OCSP/Responder URI
* Certificate KeyUsage validation
* Certificate ExtendedKeyUsage validation

#### Enable X.509 Client Certificate User Authentication <a href="#enable_x_509_client_certificate_user_authentication" id="enable_x_509_client_certificate_user_authentication"></a>

The following sections describe how to configure Wildfly/Undertow and the Keycloak Server to enable X.509 client certificate authentication.

**Enable mutual SSL in WildFly**

See [Enable SSL](https://docs.jboss.org/author/display/WFLY10/Admin+Guide#AdminGuide-EnableSSL) and [SSL](https://docs.jboss.org/author/display/WFLY10/Admin+Guide#AdminGuide-%7B%7B%3Cssl%2F%3E%7D%7D) for the instructions how to enable SSL in Wildfly.

* Open $KEYCLOAK\_HOME/standalone/configuration/standalone.xml and add a new realm:

```
<security-realms>
    <security-realm name="ssl-realm">
        <server-identities>
            <ssl>
                <keystore path="servercert.jks"
                          relative-to="jboss.server.config.dir"
                          keystore-password="servercert password"/>
            </ssl>
        </server-identities>
        <authentication>
            <truststore path="truststore.jks"
                        relative-to="jboss.server.config.dir"
                        keystore-password="truststore password"/>
        </authentication>
    </security-realm>
</security-realms>
```

`ssl/keystore`

The `ssl` element contains the `keystore` element that defines how to load the server public key pair from a JKS keystore

`ssl/keystore/path`

A path to a JKS keystore

`ssl/keystore/relative-to`

Defines a path the keystore path is relative to

`ssl/keystore/keystore-password`

The password to open the keystore

`ssl/keystore/alias` (optional)

The alias of the entry in the keystore. Set it if the keystore contains multiple entries

`ssl/keystore/key-password` (optional)

The private key password, if different from the keystore password.

`authentication/truststore`

Defines how to load a trust store to verify the certificate presented by the remote side of the inbound/outgoing connection. Typically, the truststore contains a collection of trusted CA certificates.

`authentication/truststore/path`

A path to a JKS keystore that contains the certificates of the trusted CAs (certificate authorities)

`authentication/truststore/relative-to`

Defines a path the truststore path is relative to

`authentication/truststore/keystore-password`

The password to open the truststore

**Enable https listener**

See [HTTPS Listener](https://docs.jboss.org/author/display/WFLY10/Admin+Guide#AdminGuide-HTTPSlistener) for the instructions how to enable HTTPS in Wildfly.

* Add the \<https-listener> element as shown below:

```xml
<subsystem xmlns="urn:jboss:domain:undertow:3.0">
	....
    <server name="default-server">
	    <https-listener name="default"
                        socket-binding="https"
                        security-realm="ssl-realm"
                        verify-client="REQUESTED"/>
    </server>
</subsystem>
```

`https-listener/security-realm`

The value must match the name of the realm from the previous section

`https-listener/verify-client`

If set to `REQUESTED`, the server will optionally ask for a client certificate. Setting the attribute to `REQUIRED` will have the server to refuse inbound connections if no client certificate has been provided.

#### Adding X.509 Client Certificate Authentication to a Browser Flow <a href="#adding_x_509_client_certificate_authentication_to_a_browser_flow" id="adding_x_509_client_certificate_authentication_to_a_browser_flow"></a>

* Select a realm, click on Authentication link, select the "Browser" flow
* Make a copy of the buit-in "Browser" flow. You may want to give the new flow a distinctive name, i.e. "X.509 Browser"
* Using the drop down, select the copied flow, and click on "Add Execution"
* Select "X509/Validate User Form" using the drop down and click on "Save"

![x509-execution.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/x509-execution.png)

* Using the up/down arrows, change the order of the "X509/Validate Username Form" by moving it above the "Browser Forms" execution, and set the requirement to "ALTERNATIVE"

![x509-browser-flow.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/x509-browser-flow.png)

**Configuring X.509 Client Certificate Authentication**

![x509-configuration.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/x509-configuration.png)

`User Identity Source`

Defines how to extract the user identity from a client certificate.

`A regular expression` (optional)

Defines a regular expression to use as a filter to extract the certificate identity. The regular expression must contain a single group.

`User Mapping Method`

Defines how to match the certificate identity to an existing user. _Username or e-mail_ will search for an existing user by username or e-mail. _Custom Attribute Mapper_ will search for an existing user with a custom attribute which value matches the certificate identity. The name of the custom attribute is configurable.

`A name of user attribute` (optional)

A custom attribute which value will be matched against the certificate identity.

`CRL Checking Enabled` (optional)

Defines whether to check the revocation status of the certificate using Certificate Revocation List.

`Enable CRL Distribution Point to check certificate revocation status` (optional)

Defines whether to use CDP to check the certificate revocation status. Most PKI authorities include CDP in their certificates.

`CRL file path` (optional)

Defines a path to a file that contains a CRL list. The value must be a path to a valid file if `CRL Checking Enabled` option is turned on.

`OCSP Checking Enabled`(optional)

Defines whether to check the certificate revocation status using Online Certificate Status Protocol.

`OCSP Responder URI` (optional)

Allows to override a value of the OCSP responder URI in the certificate.

`Validate Key Usage` (optional)

Verifies whether the certificate’s KeyUsage extension bits are set. For example, "digitalSignature,KeyEncipherment" will verify if bits 0 and 2 in the KeyUsage extension are asserted. Leave the parameter empty to disable the Key Usage validaion. See [RFC5280, Section-4.2.1.3](https://tools.ietf.org/html/rfc5280#section-4.2.1.3)

`Validate Extended Key Usage` (optional)

Verifies one or more purposes as defined in the Extended Key Usage extension. See [RFC5280, Section-4.2.1.12](https://tools.ietf.org/html/rfc5280#section-4.2.1.12). Leave the parameter empty to disable the Extended Key Usage validation.

`Bypass identity confirmation`

If set, X.509 client certificate authentication will not prompt the user to confirm the certificate identity and will automatiocally sign in the user upon successful authentication.

#### Adding X.509 Client Certificate Authentication to a Direct Grant Flow <a href="#adding_x_509_client_certificate_authentication_to_a_direct_grant_flow" id="adding_x_509_client_certificate_authentication_to_a_direct_grant_flow"></a>

* Using keycloak admin console, click on "Authentication" and select the "Direct Grant" flow,
* Make a copy of the build-in "Direct Grant" flow. You may want to give the new flow a distinctive name, i.e. "X509 Direct Grant",
* Delete "Validate Username" and "Password" authenticators,
* Click on "Execution" and add "X509/Validate Username" and click on "Save" to add the execution step to the parent flow.

![x509-directgrant-execution.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/x509-directgrant-execution.png)

* Change the `Requirement` to _REQUIRED_.

![x509-directgrant-flow.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/x509-directgrant-flow.png)

* Set up the x509 authentication configuration by following the steps described earlier in the x.509 Browser Flow section.

#### Troubleshooting <a href="#troubleshooting" id="troubleshooting"></a>

**Direct Grant authentication with X.509**

The following template can be used to request a token using the Resource Owner Password Credentials Grant:

```
$ curl https://[host][:port]/auth/realms/master/protocol/openid-connect/token \
       --insecure \
       --data "grant_type=password&scope=openid profile&username=&password=&client_id=CLIENT_ID&client_secret=CLIENT_SECRET" \
       -E /path/to/client_cert.crt \
       --key /path/to/client_cert.key
```

`[host][:port]`

The host and the port number of a remote Keycloak server that has been configured to allow users authenticate with x.509 client certificates using the Direct Grant Flow.

`CLIENT_ID`

A client id.

`CLIENT_SECRET`

For confidential clients, a client secret; otherwise, leave it empty.

`client_cert.crt`

A public key certificate that will be used to verify the identity of the client in mutual SSL authentication. The certificate should be in PEM format.

`client_cert.key`

A private key in the public key pair. Also expected in PEM format.
