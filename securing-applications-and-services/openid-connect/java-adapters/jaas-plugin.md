# JAAS plugin

It’s generally not needed to use JAAS for most of the applications, especially if they are HTTP based, and you should most likely choose one of our other adapters. However, some applications and systems may still rely on pure legacy JAAS solution. Keycloak provides two login modules to help in these situations.

The provided login modules are:

org.keycloak.adapters.jaas.DirectAccessGrantsLoginModule

This login module allows to authenticate with username/password from Keycloak. It’s using [Resource Owner Password Credentials](https://wjw465150.gitbooks.io/keycloak-documentation/content/securing\_apps/topics/oidc/oidc-generic.html#\_resource\_owner\_password\_credentials\_flow) flow to validate if the provided username/password is valid. It’s useful for non-web based systems, which need to rely on JAAS and want to use Keycloak, but can’t use the standard browser based flows due to their non-web nature. Example of such application could be messaging or SSH.

org.keycloak.adapters.jaas.BearerTokenLoginModule

This login module allows to authenticate with Keycloak access token passed to it through CallbackHandler as password. It may be useful for example in case, when you have Keycloak access token from standard based authentication flow and your web application then needs to talk to external non-web based system, which rely on JAAS. For example a messaging system.

Both modules use the following configuration properties:

keycloak-config-file

The location of the `keycloak.json` configuration file. The configuration file can either be located on the filesystem or on the classpath. If it’s located on the classpath you need to prefix the location with `classpath:` (for example `classpath:/path/keycloak.json`). This is _REQUIRED._

`role-principal-class`

Configure alternative class for Role principals attached to JAAS Subject. Default value is `org.keycloak.adapters.jaas.RolePrincipal`. Note: The class is required to have a constructor with a single `String` argument.
