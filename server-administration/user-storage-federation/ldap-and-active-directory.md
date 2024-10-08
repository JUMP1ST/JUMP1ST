# LDAP and Active Directory

Keycloak comes with a built-in LDAP/AD provider. It is possible to federate multiple different LDAP servers in the same Keycloak realm. You can map LDAP user attributes into the Keycloak common user model. By default, it maps username, email, first name, and last name, but you are free to configure additional [mappings](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/user-federation/ldap.html#\_ldap\_mappers). The LDAP provider also supports password validation via LDAP/AD protocols and different storage, edit, and synchronization modes.

To configure a federated LDAP store go to the Admin Console. Click on the `User Federation` left menu option. When you get to this page there is an `Add Provider` select box. You should see _ldap_ within this list. Selecting _ldap_ will bring you to the ldap configuration page.

**Storage Mode**

By default, Keycloak will import users from LDAP into the local Keycloak user database. This copy of the user is either synchronized on demand, or through a periodic background task. The one exception to this is passwords. Passwords are not imported and password validation is delegated to the LDAP server. The benefits to this approach is that all Keycloak features will work as any extra per-user data that is needed can be stored locally. This approach also reduces load on the LDAP server as uncached users are loaded from the Keycloak database the 2nd time they are accessed. The only load your LDAP server will have is password validation. The downside to this approach is that when a user is first queried, this will require a Keycloak database insert. The import will also have to be synchronized with your LDAP server as needed.

Alternatively, you can choose not to import users into the Keycloak user database. In this case, the common user model that the Keycloak runtime uses is backed only by the LDAP server. This means that if LDAP doesn’t support a piece of data that a Keycloak feature needs that feature will not work. The benefit to this approach is that you do not have the overhead of importing and synchronizing a copy of the LDAP user into the Keycloak user database.

This storage mode is controled by the `Import Enabled` switch. Set to `On` to import users.

**Edit Mode**

Users, through the [User Account Service](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/account.html#\_account-service), and admins through the Admin Console have the ability to modify user metadata. Depending on your setup you may or may not have LDAP update privileges. The `Edit Mode` configuration option defines the edit policy you have with your LDAP store.

READONLY

Username, email, first name, last name, and other mapped attributes will be unchangeable. Keycloak will show an error anytime anybody tries to update these fields. Also, password updates will not be supported.

WRITABLE

Username, email, first name, last name, and other mapped attributes and passwords can all be updated and will be synchronized automatically with your LDAP store.

UNSYNCED

Any changes to username, email, first name, last name, and passwords will be stored in Keycloak local storage. It is up to you to figure out how to synchronize back to LDAP. This allows Keycloak deployments to support updates of user metadata on a read-only LDAP server. This option only applies when you are importing users from LDAP into the local Keycloak user database.

**Other config options**

Console Display Name

Name used when this provider is referenced in the admin console

Priority

The priority of this provider when looking up users or for adding registrations.

Sync Registrations

Does your LDAP support adding new users? Click this switch if you want new users created by Keycloak in the admin console or the registration page to be added to LDAP.

Allow Kerberos authentication

Enable Kerberos/SPNEGO authentication in realm with users data provisioned from LDAP. More info in [Kerberos section](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/authentication/kerberos.html#\_kerberos).

Other options

The rest of the configuration options should be self explanatory. You can mouseover the tooltips in Admin Console to see some more details about them.

**Connect to LDAP over SSL**

When you configure a secured connection URL to your LDAP store(for example `ldaps://myhost.com:636` ), Keycloak will use SSL for the communication with LDAP server. The important thing is to properly configure a truststore on the Keycloak server side, otherwise Keycloak can’t trust the SSL connection to LDAP.

The global truststore for the Keycloak can be configured with the Truststore SPI. Please check out the [Server Installation and Configuration](https://keycloak.gitbooks.io/documentation/content/server\_installation/index.html) for more detail. If you don’t configure the truststore SPI, the truststore will fallback to the default mechanism provided by Java (either the file provided by system property `javax.net.ssl.trustStore` or the cacerts file from the JDK if the system property is not set).

There is a configuration property `Use Truststore SPI` in the LDAP federation provider configuration, where you can choose whether the Truststore SPI is used. By default, the value is `Only for ldaps`, which is fine for most deployments. The Truststore SPI will only be used if the connection to LDAP starts with `ldaps`.

**Sync of LDAP users to Keycloak**

If you have import enabled, the LDAP Provider will automatically take care of synchronization (import) of needed LDAP users into the Keycloak local database. As users log in, the LDAP provider will import the LDAP user into the Keycloak database and then authenticate against the LDAP password. This is the only time users will be imported. If you go to the `Users` left menu item in the Admin Console and click the `View all users` button, you will only see those LDAP users that have been authenticated at least once by Keycloak. It is implemented this way so that admins don’t accidentally try to import a huge LDAP DB of users.

If you want to sync all LDAP users into the Keycloak database, you may configure and enable the `Sync Settings` of the LDAP provider you configured. There are 2 types of synchronization:

Periodic Full sync

This will synchronize all LDAP users into Keycloak DB. Those LDAP users, which already exist in Keycloak and were changed in LDAP directly will be updated in Keycloak DB (For example if user `Mary Kelly` was changed in LDAP to `Mary Smith`).

Periodic Changed users sync

When syncing occurs, only those users that were created or updated after the last sync will be updated and/or imported.

The best way to handle syncing is to click the `Synchronize all users` button when you first create the LDAP provider, then set up a periodic sync of changed users. The configuration page for your LDAP Provider has several options to support you.

**LDAP Mappers**

LDAP mappers are `listeners`, which are triggered by the LDAP Provider at various points, provide another extension point to LDAP integration. They are triggered when a user logs in via LDAP and needs to be imported, during Keycloak initiated registration, or when a user is queried from the Admin Console. When you create an LDAP Federation provider, Keycloak will automatically provide set of built-in `mappers` for this provider. You are free to change this set and create a new mapper or update/delete existing ones.

User Attribute Mapper

This allows you to specify which LDAP attribute is mapped to which attribute of Keycloak user. So, for example, you can configure that LDAP attribute `mail` to the attribute `email` in the Keycloak database. For this mapper implementation, there is always a one-to-one mapping (one LDAP attribute is mapped to one Keycloak attribute)

FullName Mapper

This allows you to specify that the full name of the user, which is saved in some LDAP attribute (usually `cn` ) will be mapped to `firstName` and `lastname` attributes in the Keycloak database. Having `cn` to contain full name of user is a common case for some LDAP deployments.

Role Mapper

This allows you to configure role mappings from LDAP into Keycloak role mappings. One Role mapper can be used to map LDAP roles (usually groups from a particular branch of LDAP tree) into roles corresponding to either realm roles or client roles of a specified client. It’s not a problem to configure more Role mappers for the same LDAP provider. So for example you can specify that role mappings from groups under `ou=main,dc=example,dc=org` will be mapped to realm role mappings and role mappings from groups under `ou=finance,dc=example,dc=org` will be mapped to client role mappings of client `finance` .

Hardcoded Role Mapper

This mapper will grant a specified Keycloak role to each Keycloak user linked with LDAP.

Group Mapper

This allows you to configure group mappings from LDAP into Keycloak group mappings. Group mapper can be used to map LDAP groups from a particular branch of an LDAP tree into groups in Keycloak. It will also propagate user-group mappings from LDAP into user-group mappings in Keycloak.

MSAD User Account Mapper

This mapper is specific to Microsoft Active Directory (MSAD). It’s able to tightly integrate the MSAD user account state into the Keycloak account state (account enabled, password is expired etc). It’s using the `userAccountControl` and `pwdLastSet` LDAP attributes. (both are specific to MSAD and are not LDAP standard). For example if `pwdLastSet` is `0`, the Keycloak user is required to update their password and there will be an UPDATE\_PASSWORD required action added to the user. If `userAccountControl` is `514` (disabled account) the Keycloak user is disabled as well.

By default, there is set of User Attribute mappers that map basic Keycloak user attributes like username, first name, lastname, and email to corresponding LDAP attributes. You are free to extend these and provide additional attribute mappings. Admin console provides tooltips, which should help with configuring the corresponding mappers.
