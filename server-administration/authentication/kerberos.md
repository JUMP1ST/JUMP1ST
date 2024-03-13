# Kerberos

Keycloak supports login with a Kerberos ticket through the SPNEGO protocol. SPNEGO (Simple and Protected GSSAPI Negotiation Mechanism) is used to authenticate transparently through the web browser after the user has been authenticated when logging-in his session. For non-web cases or when ticket is not available during login, Keycloak also supports login with Kerberos username/password.

A typical use case for web authentication is the following:

1. User logs into his desktop (Such as a Windows machine in Active Directory domain or Linux machine with Kerberos integration enabled).
2. User then uses his browser (IE/Firefox/Chrome) to access a web application secured by Keycloak.
3. Application redirects to Keycloak login.
4. Keycloak renders HTML login screen together with status 401 and HTTP header `WWW-Authenticate: Negotiate`
5. In case that the browser has Kerberos ticket from desktop login, it transfers the desktop sign on information to the Keycloak in header `Authorization: Negotiate 'spnego-token'` . Otherwise it just displays the login screen.
6. Keycloak validates token from the browser and authenticates the user. It provisions user data from LDAP (in case of LDAPFederationProvider with Kerberos authentication support) or let user to update his profile and prefill data (in case of KerberosFederationProvider).
7. Keycloak returns back to the application. Communication between Keycloak and application happens through OpenID Connect or SAML messages. The fact that Keycloak was authenticated through Kerberos is hidden from the application. So Keycloak acts as broker to Kerberos/SPNEGO login.

For setup there are 3 main parts:

1. Setup and configuration of Kerberos server (KDC)
2. Setup and configuration of Keycloak server
3. Setup and configuration of client machines

**Setup of Kerberos server**

This is platform dependent. Exact steps depend on your OS and the Kerberos vendor you’re going to use. Consult Windows Active Directory, MIT Kerberos and your OS documentation for how exactly to setup and configure Kerberos server.

At least you will need to:

* Add some user principals to your Kerberos database. You can also integrate your Kerberos with LDAP, which means that user accounts will be provisioned from LDAP server.
*   Add service principal for "HTTP" service. For example if your Keycloak server will be running on `www.mydomain.org` you may need to add principal `HTTP/`[`[email protected]`](https://wjw465150.gitbooks.io/cdn-cgi/l/email-protection) assuming that MYDOMAIN.ORG will be your Kerberos realm.

    For example on MIT Kerberos you can run a "kadmin" session. If you are on the same machine where is MIT Kerberos, you can simply use the command:

```
sudo kadmin.local
```

Then add HTTP principal and export his key to a keytab file with the commands like:

```
addprinc -randkey HTTP/
ktadd -k /tmp/http.keytab HTTP/
```

The Keytab file `/tmp/http.keytab` will need to be accessible on the host where Keycloak server will be running.

**Setup and configuration of Keycloak server**

You need to install a kerberos client on your machine. This is also platform dependent. If you are on Fedora, Ubuntu or RHEL, you can install the package `freeipa-client`, which contains a Kerberos client and several other utilities. Configure the kerberos client (on linux it’s in file `/etc/krb5.conf` ). You need to put your Kerberos realm and at least configure the HTTP domains your server will be running on. For the example realm MYDOMAIN.ORG you may configure the `domain_realm` section like this:

```
[domain_realm]
  .mydomain.org = MYDOMAIN.ORG
  mydomain.org = MYDOMAIN.ORG
```

Next you need to export the keytab file with the HTTP principal and make sure the file is accessible to the process under which Keycloak server is running. For production, it’s ideal if it’s readable just by this process and not by someone else. For the MIT Kerberos example above, we already exported keytab to `/tmp/http.keytab` . If your KDC and Keycloak are running on same host, you have that file already available.

**Enable SPNEGO Processing**

Keycloak does not have the SPNEGO protocol support turned on by default. So, you have to go to the [browser flow](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/authentication/flows.html#\_authentication-flows) and enable `Kerberos`.

Browser Flow

![browser-flow.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/browser-flow.png)

Switch the `Kerberos` requirement from _disabled_ to either _alternative_ or _required_. _Alternative_ basically means that Kerberos is optional. If the user’s browser hasn’t been configured to work with SPNEGO/Kerberos, then Keycloak will fall back to the regular login screens. If you set the requirement to _required_ then all users must have Kerberos enabled for their browser.

**Configure Kerberos User Storage Federation Provider**

Now that the SPNEGO protocol is turned on at the authentication server, you’ll need to configure how Keycloak interprets the Kerberos ticket. This is done through [User Storage Federation](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/user-federation.html#\_user-storage-federation). We have 2 different federation providers with Kerberos authentication support.

If you want to authenticate with Kerberos backed by an LDAP server, you have to first configure the [LDAP Federation Provider](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/user-federation/ldap.html#\_ldap). If you look at the configuration page for your LDAP provider you’ll see a `Kerberos Integration` section.

LDAP Kerberos Integration

![ldap-kerberos.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/ldap-kerberos.png)

Turning on the switch `Allow Kerberos authentication` will make Keycloak use the Kerberos principal to lookup information about the user so that it can be imported into the Keycloak environment.

If your Kerberos solution is not backed by an LDAP server, you have to use the `Kerberos` User Storage Federation Provider. Go to the `User Federation` left menu item and select `Kerberos` from the `Add provider` select box.

Kerberos User Storage Provider

![kerberos-provider.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/kerberos-provider.png)

This provider parses the Kerberos ticket for simple principal information and does a small import into the local Keycloak database. User profile information like first name, last name, and email are not provisioned.

**Setup and configuration of client machines**

Clients need to install kerberos client and setup krb5.conf as described above. Additionally they need to enable SPNEGO login support in their browser. See [configuring Firefox for Kerberos](http://www.microhowto.info/howto/configure\_firefox\_to\_authenticate\_using\_spnego\_and\_kerberos.html) if you are using that browser. URI `.mydomain.org` must be allowed in the `network.negotiate-auth.trusted-uris` config option.

In a Windows domain, clients usually don’t need to configure anything special as IE is already able to participate in SPNEGO authentication for the Windows domain.

**Example setups**

For easier testing with Kerberos, we provided some example setups to test.

**Keycloak and FreeIPA docker image**

Once you install [docker](https://www.docker.com/), you can run docker image with [FreeIPA](http://www.freeipa.org/) server installed. FreeIPA provides integrated security solution with MIT Kerberos and 389 LDAP server among other things . The image provides also Keycloak server configured with LDAP Federation provider and enabled SPNEGO/Kerberos authentication against the FreeIPA server. See details [here](https://github.com/mposolda/keycloak-freeipa-docker/blob/master/README.md) .

**ApacheDS testing Kerberos server**

For quick testing and unit tests, we use a very simple [ApacheDS](http://directory.apache.org/apacheds/) Kerberos server. You need to build Keycloak from sources and then run the Kerberos server with maven-exec-plugin from our testsuite. See details [here](https://github.com/keycloak/keycloak/blob/master/misc/Testsuite.md#kerberos-server) .

**Credential Delegation**

Kerberos 5 supports the concept of credential delegation. In this scenario, your applications may want access to the Kerberos ticket so that they can re-use it to interact with other services secured by Kerberos. Since the SPNEGO protocol is processed in the Keycloak server, you have to propagate the GSS credential to your application within the OpenID Connect token claim or a SAML assertion attribute that is transmitted to your application from the Keycloak server. To have this claim inserted into the token or assertion, each application will need to enable the built-in protocol mapper called `gss delegation credential`. This is enabled in the `Mappers` tab of the application’s client page. See [Protocol Mappers](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/clients/protocol-mappers.html#\_protocol-mappers) chapter for more details.

Applications will need to deserialize the claim it receives from Keycloak before it can use it to make GSS calls against other services. Once you deserialize the credential from the access token to the GSSCredential object, the GSSContext will need to be created with this credential passed to the method `GSSManager.createContext` for example like this:

```
// Obtain accessToken in your application.
KeycloakPrincipal keycloakPrincipal = (KeycloakPrincipal) servletReq.getUserPrincipal();
AccessToken accessToken = keycloakPrincipal.getKeycloakSecurityContext().getToken();

// Retrieve kerberos credential from accessToken and deserialize it
String serializedGssCredential = (String) accessToken.getOtherClaims().
    get(org.keycloak.common.constants.KerberosConstants.GSS_DELEGATION_CREDENTIAL);

GSSCredential deserializedGssCredential = org.keycloak.common.util.KerberosSerializationUtils.
    deserializeCredential(serializedGssCredential);

// Create GSSContext to call other kerberos-secured services
GSSContext context = gssManager.createContext(serviceName, krb5Oid,
    deserializedGssCredential, GSSContext.DEFAULT_LIFETIME);
```

We have an example, that shows this in detail. It’s in `examples/kerberos` in the Keycloak example distribution or demo distribution download. You can also check the example sources directly [here](https://github.com/keycloak/keycloak/blob/master/examples/kerberos) .

Note that you also need to configure `forwardable` kerberos tickets in `krb5.conf` file and add support for delegated credentials to your browser.

| Warning | Credential delegation has some security implications so only use it if you really need it. It’s highly recommended to use it together with HTTPS. See for example [this article](http://www.microhowto.info/howto/configure\_firefox\_to\_authenticate\_using\_spnego\_and\_kerberos.html#idp27072) for more details. |
| ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

**Troubleshooting**

If you have issues, we recommend that you enable additional logging to debug the problem:

* Enable `Debug` flag in admin console for Kerberos or LDAP federation providers
* Enable TRACE logging for category `org.keycloak` in logging section of `standalone/configuration/standalone.xml` to receive more info `standalone/log/server.log`
* Add system properties `-Dsun.security.krb5.debug=true` and `-Dsun.security.spnego.debug=true`
