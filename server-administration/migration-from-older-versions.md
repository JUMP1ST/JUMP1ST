# Migration from older versions

o upgrade to a new version of Keycloak first download and install the new version of Keycloak. Once the new version is installed migrate the config files, database, keycloak-server.json, providers, themes and applications to the new applications.

This chapter contains some general migration details which are applicable to all versions. There are instructions for migration that is only applicable to a specific release. If you are upgrading from a very old version you need to go through all intermediate version specific migration.

It’s highly recommended that you backup your database prior to upgrading Keycloak.

Migration from a candidate release (CR) to a Final release is not supported. We do however recommend that you test migration for a CR so we can resolve any potential issues before the Final is released.

#### Migration of config files using migration scripts <a href="#migration_of_config_files_using_migration_scripts" id="migration_of_config_files_using_migration_scripts"></a>

As part of your migration, you should use your old versions of config files `standalone.xml`, `standalone-ha.xml`, and/or `domain.xml`. These files typically contain configuration that is unique to your own environment. So, the first thing to do as part of a migration is to copy those files to the new Keycloak server installation, replacing the default versions.

If migrating from Keycloak version 2.1.0 or older, you should also copy `keycloak-server.json` to `standalone/configuration` and/or `domain/configuration`.

There will be configuration in those config files that pertains to Keycloak, which will need to be upgraded. For that, you should run one or more of the appropriate upgrade scripts. They are `migrate-standalone.cli`, `migrate-standalone-ha.cli`, `migrate-domain-standalone.cli` and `migrate-domain-clustered.cli`.

The server should not be running when you execute a migration script.

Example for running migrate-standalone.cli

```
$ .../bin/jboss-cli.sh --file=migrate-standalone.cli
```

Note that for upgrading domain.xml, there are two migration scripts, `migrate-domain-standalone.cli` and `migrate-domain-clustered.cli`. These scripts migrate separate profiles within your domain.xml that were originally shipped with Keycloak. If you have changed the name of the profile there is a variable near the top of the script that you will need to change.

If you are migrating `keycloak-server.json`, this will also be migrated as needed. If you prefer, you can migrate `keycloak-server.json` beforehand using the instructions in the next section.

One thing to note is that the migration scripts only work for Keycloak versions 1.8.1 forward. If migrating an older version you will need to manually upgrade your config files to at least be 1.8.1 compliant.

Lastly, you may want to examine the contents of the scripts before running. They show exactly what will be changed for each version. They also have values at the top of the script that you may need to change based on your environment.

#### Migrate and convert keycloak-server.json <a href="#migrate_and_convert_keycloak_server_json" id="migrate_and_convert_keycloak_server_json"></a>

If you ran one of the migration scripts from the previous section then you have probably already migrated your keycloak-server.json. This section is kept here for reference. If you prefer, you can follow the instructions in this section before running the migration scripts above.

You should copy `standalone/configuration/keycloak-server.json` from the old version to make sure any configuration changes you’ve done are added to the new installation. The version specific section below will list any changes done to this file that you have to do when upgrading from one version to another.

Keycloak is moving away from the use of keycloak-server.json. For this release, the server will still work if this file is in `standalone/configuration/keycloak-server.json`, but it is highly recommended that you convert to using standalone.xml, standalone-ha.xml, or domain.xml for configuration. We may soon remove support for keycloak-server.json.

To convert your keycloak-server.json, you will use a jboss-cli operation called `migrate-json`. It is recommended that you run this operation while the server is not running.

The `migrate-json` operation assumes you are migrating with an xml configuration file from an old version. For example, you should not try to migrate your keycloak-server.json to a standalone.xml file that already contains \<spi> and \<theme> tags under the keycloak subsystem. Before migration, your keycloak subsystem should look like the one below:

standalone.xml

```xml
<subsystem xmlns="urn:jboss:domain:keycloak-server:1.1">
    <web-context>auth</web-context>
</subsystem>
```

The jboss-cli tool is discussed in detail in [Server Installation and Configuration](https://keycloak.gitbooks.io/documentation/content/server\_installation/index.html).

**migrate-json in Standalone Mode**

For standalone, you will issue the `migrate-json` operation in `embed` mode without the server running.

Standalone keycloak-server.json migration

```
$ .../bin/jboss-cli.sh
[disconnected /] embed-server --server-config=standalone.xml
[standalone@embedded /] /subsystem=keycloak-server/:migrate-json
```

The `migrate-json` operation will look for your keycloak-server.json file in the `standalone/configuration` directory. You also have the option of using the `file` argument as shown in the domain mode example below.

**migrate-json in Domain Mode**

For a domain, you will stop the Keycloak server and issue the `migrate-json` operation against the running domain controller. If you choose not to stop the Keycloak server, the operation will still work, but your changes will not take affect until the Keycloak server is restarted.

Domain mode migration requires that you use the `file` parameter to upload your keycloak-server.json from a local directory. The example below shows connecting to localhost. You will need to substitute the address of your domain controller.

Domain mode keycloak-server.json migration

```
$ .../bin/jboss-cli.sh -c --controller=localhost:9990
[domain@localhost:9990 /] cd profile=auth-server-clustered
[domain@localhost:9990 profile=auth-server-clustered] cd subsystem=keycloak-server
[domain@localhost:9990 subsystem=keycloak-server] :migrate-json(file="./keycloak-server.json")
```

You will need to repeat the `migrate-json` operation for each profile containing a `keycloak-server` subsystem.

#### Migrate database <a href="#migrate_database" id="migrate_database"></a>

Keycloak can automatically migrate the database schema, or you can choose to do it manually.

**Relational database**

To enable automatic upgrading of the database schema set the `migrationStrategy` property to `update` for the default `connectionsJpa` provider:

Edit xml

```xml
<spi name="connectionsJpa">
    <provider name="default" enabled="true">
        <properties>
            ...
            <property name="migrationStrategy" value="update"/>
        </properties>
    </provider>
</spi>
```

Equivalent CLI command for above

```
/subsystem=keycloak-server/spi=connectionsJpa/provider=default/:map-put(name=properties,key=migrationStrategy,value=update)
```

When you start the server with this setting your database will automatically be migrated if the database schema has changed in the new version.

To manually migrate the database set the `migrationStrategy` to `manual`. When you start the server with this configuration it will check if the database needs migration. If changes are needed the required changes are written to an SQL file that you can review and manually run against the database.

There’s also the option to disable migration by setting the `migrationStrategy` to `validate`. With this configuration the database will be checked at startup and if it is not migrated the server will exit.

**Mongo**

Mongo doesn’t have a schema, but there may still be things like collections and indexes that are added to new releases. To enable automatic creation of these set the `migrationStrategy` property to `update` for the default `connectionsMongo` provider:

Edit xml

```xml
<spi name="connectionsMongo">
    <provider name="default" enabled="true">
        <properties>
            ...
            <property name="migrationStrategy" value="update"/>
        </properties>
    </provider>
</spi>
```

Equivalent CLI command for above

```
/subsystem=keycloak-server/spi=connectionsMongo/provider=default/:map-put(name=properties,key=migrationStrategy,value=update)
```

The Mongo provider does not have the option to manually apply the required changes.

There’s also the option to disable migration by setting the `migrationStrategy` to `validate`. With this configuration the database will be checked at startup and if it is not migrated the server will exit.

#### Migrate providers <a href="#migrate_providers" id="migrate_providers"></a>

If you have implemented any SPI providers you need to copy them to the new server. The version specific section below will mention if any of the SPI’s have changed. If they have you may have to update your code accordingly.

#### Migrate themes <a href="#migrate_themes" id="migrate_themes"></a>

If you have created a custom theme you need to copy them to the new server. The version specific section below will mention if changes have been made to themes. If there is you may have to update your themes accordingly.

#### Migrate application <a href="#migrate_application" id="migrate_application"></a>

If you deploy applications directly to the Keycloak server you should copy them to the new server. For any applications including those not deployed directly to the Keycloak server you should upgrade the adapter. The version specific section below will mention if any changes are required to applications.

#### Version specific migration <a href="#version_specific_migration" id="version_specific_migration"></a>

**Migrating to 3.2.0**

**New Password Hashing algorithms**

We’ve added two new password hashing algorithms (pbkdf2-sha256 and pbkdf2-sha512). New realms will use the pbkdf2-sha256 hashing algorithm with 27500 hashing iterations. Since pbkdf2-sha256 is slightly faster than pbkdf2 the iterations was increased to 27500 from 20000.

Existing realms are upgraded if the password policy contains the default value for hashing algorithm (not specified) and iteration (20000). If you have changed the hashing iterations you need to manually change to pbkdf2-sha256 if you’d like to use the more secure hashing algorithm.

**ID Token requires scope=openid**

OpenID Connect specification requires that parameter `scope` with value `openid` is used in initial authorization request. So in Keycloak 2.1.0 we changed our adapters to use `scope=openid` in the redirect URI to Keycloak. Now we changed the server part too and ID token will be sent to the application just if `scope=openid` is really used. If it’s not used, then ID token will be skipped and just Access token and Refresh token will be sent to the application. This also allows that you can ommit the generation of ID Token on purpose - for example for space or performance purposes.

Direct grants (OAuth2 Resource Owner Password Credentials Grant) and Service accounts login (OAuth2 Client credentials grant) also don’t use ID Token by default now. You need to explicitly add `scope=openid` parameter to have ID Token included.

**Authentication sessions and Action tokens**

We are working on support for multiple datacenters. As the initial step, we introduced authentication session and action tokens. Authentication session replaces Client session, which was used in previous versions. Action tokens are currently used especially for the scenarios, where the authenticator or requiredActionProvider requires sending email to the user and requires user to click on the link in email.

Here are concrete changes related to this, which may affect you for the migration.

First change related to this is introducing new infinispan caches `authenticationSessions` and `actionTokens` in `standalone.xml` or `standalone-ha.xml`. If you use our migration CLI, you don’t need to care much as your `standalone(-ha).xml` files will be migrated automatically.

Second change is changing of some signatures in methods of authentication SPI. This may affect you if you use custom `Authenticator` or `RequiredActionProvider`. Classes `AuthenticationFlowContext` and `RequiredActionContext` now allow to retrieve authentication session instead of client session.

We also added some initial support for sticky sessions. You may want to change your loadbalancer/proxy server and configure it if you want to suffer from it and have better performance. The route is added to the new `AUTH_SESSION_ID` cookie. More info in the clustering documentation.

Another change is, that `token.getClientSession()` was removed. This may affect you for example if you’re using Client Initiated Identity Broker Linking feature.

The `ScriptBasedAuthenticator` changed the binding name from `clientSession` to `authenticationSession`, so you would need to update your scripts if you’re using this authenticator.

Finally we added some new timeouts to the admin console. This allows you for example to specify different timeouts for the email actions triggered by admin and by user himself.

**Migrating to 2.5.1**

**Migration of old offline tokens**

If you migrate from version 2.2.0 or older and you used offline tokens, then your offline tokens didn’t have KID in the token header. We added KID to the token header in 2.3.0 together with the ability to have multiple realm keys, so Keycloak is able to find the correct key based on the token KID.

For the offline tokens without KID, Keycloak 2.5.1 will always use the active realm key to find the proper key for the token verification. In other words, migration of old offline tokens will work. So for example, your user requested offline token in 1.9.8, then you migrate from 1.9.8 to 2.5.1 and then your user will be still able to refresh his old offline token in 2.5.1 version.

But there is limitation, that once you change the realm active key, the users won’t be able to refresh old offline tokens anymore. So you shouldn’t change the active realm key until all your users with offline tokens refreshed their tokens. Obviously newly refreshed tokens will have KID in the header, so after all users exchange their old offline tokens, you are free to change the active realm key.

**Migrating to 2.5.0**

**Changes to the infinispan caches**

The `realms` cache defined in the infinispan subsystem in `standalone.xml` or `standalone-ha.xml` configuration file, now has the eviction with the 10000 records by default. This is the same default like the `users` cache.

Also the `authorization` cache now doesn’t have any eviction on it by default.

**Migrating to 2.4.0**

**Server SPI split into Server SPI and Sever SPI Private**

The keycloak-server-spi module has been split into keycloak-server-spi and keycloak-server-spi-private. APIs within keycloak-server-spi will not change between minor releases, while we reserve the right and may quite likely change APIs in keycloak-server-spi-private between minor releases.

**Key encryption algorithm in SAML assertions**

Key in SAML assertions and documents are now encrypted using RSA-OAEP encryption scheme. If you want to use encrypted assertions, make sure that service providers understand this encryption scheme. In the unlikely case that SP would not be able to handle the new scheme, Keycloak can be made to use legacy RSA-v1.5 encryption scheme when started with system property `keycloak.saml.key_trans.rsa_v1.5` set to `true`.

**Infinispan caches realms and users are always local**

Even if you use Keycloak in cluster, the caches `realms` and `users` defined in infinispan subsystem in `standalone-ha.xml` are always local caches now. There is separate cache `work`, which handles sending invalidation messages between cluster nodes and informing whole cluster what records in underlying `realms` and `users` caches should be invalidated.

**Migrating to 2.3.0**

**Default max results on paginated endpoints**

All Admin REST API endpoints that support pagination now have a default max results set to 100. If you want to return more than 100 entries you need to explicitly specify that with `?max=<RESULTS>`.

**`realm-public-key` adapter property not recommended**

In 2.3.0 release we added support for Public Key Rotation. When admin rotates the realm keys in Keycloak admin console, the Client Adapter will be able to recognize it and automatically download new public key from Keycloak. However this automatic download of new keys is done just if you don’t have `realm-public-key` option in your adapter with the hardcoded public key. For this reason, we don’t recommend to use `realm-public-key` option in adapter configuration anymore.

Note this option is still supported, but it may be useful just if you really want to have hardcoded public key in your adapter configuration and never download the public key from Keycloak. In theory, one reason for this can be to avoid man-in-the-middle attack if you have untrusted network between adapter and Keycloak, however in that case, it is much better option to use HTTPS, which will secure all the requests between adapter and Keycloak.

**Added infinispan cache `keys`**

In this release, we added new cache `keys` to the infinispan subsystem, which is defined in `standalone.xml` or `standalone-ha.xml` configuration file. It has also some eviction and expiration defined. This cache is internally used for caching the external public keys of the entities trusted by the server (Identity providers or clients, which uses authentication with signed JWT).

**Migrating to 2.2.0**

**`databaseSchema` property deprecated**

The `databaseSchema` property for both JPA and Mongo is now deprecated and has been replaced by `initializeEmpty` and `migrationStrategy`. `initializeEmpty` can bet set to `true` or `false` and controls if an empty database should be initialized. `migrationStrategy` can be set to `update`, `validate` and `manual`. `manual` is only supported for relational databases and will write an SQL file with the required changes to the database schema. Please note that for Oracle database, the created SQL file contains `SET DEFINE OFF` command understood by Oracle SQL clients. Should the script be consumed by any other client, please replace the lines with equivalent command of the tool of your choice that disables variable expansion or remove it completely if such functionality is not applicable.

**Changes in Client’s Valid Redirect URIs**

The following scenarios are affected:

* When a Valid Redirect URI with query component is saved in a Client (e.g. `http://localhost/auth?foo=bar`), `redirect_uri` in authorization request must exactly match this URI (or other registered URI in this Client).
* When a Valid Redirect URI without a query component is saved in a Client, `redirect_uri` must exactly match as well.
* Wildcards in registered Valid Redirect URIs are no longer supported when query component is present in this URI, so the `redirect_uri` needs to exactly match this saved URI as well.
* Fragments in registered Valid Redirect URIs (like `http://localhost/auth#fragment`) are no longer allowed.

**Authenticate by default removed from Identity Providers**

Identity providers no longer has an option to set it as a default authenticaton provider. Instead go to Authentication, select the `Browser` flow and configure the `Identity Provider Redirector`. It has an option to set the default identity provider.

**Migrating to 2.0.0**

**Upgrading from 1.0.0.Final no longer supported**

Upgrading from 1.0.0.Final is no longer supported. To upgrade to this version upgrade to 1.9.8.Final prior to upgrading to 2.0.0.

**Migrating to 1.9.5**

**Default password hashing interval increased to 20K**

The default password hashing interval for new realms has been increased to 20K (from 1 previously). This change will have an impact on performance when users authenticate. For example with the old default (1) it takes less than 1 ms to hash a password, but with the new default (20K) the same operation can take 50-100 ms.

**Migrating to 1.9.3**

**Add User script renamed**

The script to add admin users to Keycloak has been renamed to `add-user-keycloak`.

**Migrating to 1.9.2**

**Adapter option auth-server-url-for-backend-requests removed**

We’ve removed the option auth-server-url-for-backend-requests as there were issues in some scenarios when it was used. In more details, it was possible to access the Keycloak server from 2 different contexts (internal and external), which was causing issues in token validations etc.

If you still want to use the optimization of network, which this option provided (avoid the application to send backchannel requests through loadbalancer but send them to local Keycloak server directly) you may need to handle it at hosts configuration (DNS) level.

**Migrating to 1.9.0**

**Themes and providers directory moved**

We’ve moved the themes and providers directories from `standalone/configuration/themes` and `standalone/configuration/providers` to `themes` and `providers` respectively. If you have added custom themes and providers you need to move them to the new location. You also need to update `keycloak-server.json` as it’s changed due to this.

**Adapter Subsystems only bring in dependencies if Keycloak is on**

Previously, if you had installed our SAML or OIDC Keycloak subsystem adapters into Wildfly or JBoss EAP, we would automatically include Keycloak client jars into EVERY application irregardless if you were using Keycloak or not. These libraries are now only added to your deployment if you have Keycloak authentication turned on for that adapter (via the subsystem, or auth-method in web.xml

**Client Registration service endpoints moved**

The Client Registration service endpoints have been moved from `{realm}/clients` to `{realm}/clients-registrations`.

**Session state parameter in authentication response renamed**

In the OpenID Connect authentication response we used to return the session state as `session-state` this is not correct according to the specification and has been renamed to `session_state`.

**Deprecated OpenID Connect endpoints**

In 1.2 we deprecated a number of endpoints that where not consistent with the OpenID Connect specifications, these have now been removed. This also applies to the validate token endpoints that was replaced with the new introspect endpoint in 1.8.

**Updates to theme templates**

Feedback in template.ftl has been moved and format has changed slightly.

**Module and Source Code Re-org**

Most of our modules and source code have been consolidated into two maven modules: keycloak-server-spi and keycloak-services. SPI interfaces are in server-spi, implementations are in keycloak-services. All JPA dependent modules have been consolidated under keycloak-model-jpa. Same goes with mongo and Infinispan under modules keycloak-model-mongo and keycloak-model-infinispan.

**For adapters, session id changed after login**

To plug a security attack vector, for platforms that support it (Tomcat 8, Undertow/Wildfly, Jetty 9), the Keycloak OIDC and SAML adapters will change the session id after login. You can turn off this behavior check adapter config switches.

**SAML SP Client Adapter Changes**

Keycloak SAML SP Client Adapter now requires a specific endpoint, `/saml` to be registered with your IDP.

**Migrating to 1.8.0**

**Admin account**

In previous releases we shipped with a default admin user with a default password, this has now been removed. If you are doing a new installation of 1.8 you will have to create an admin user as a first step. This can be done easily by following the steps in [Admin User](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/MigrationFromOlderVersions.html#\_create\_admin\_user).

**OAuth2 Token Introspection**

In order to add more compliance with OAuth2 specification, we added a new endpoint for token introspection. The new endpoint can reached at `/realms/{realm}/protocols/openid-connect/token/introspect` and it is solely based on `RFC-7662.`

The `/realms/{realm}/protocols/openid-connect/validate` endpoint is now deprecated and we strongly recommend you to move to the new introspection endpoint as soon as possible. The reason for this change is that RFC-7662 provides a more standard and secure introspection endpoint.

The new token introspection URL can now be obtained from OpenID Connect Provider’s configuration at `/realms/{realm}/.well-known/openid-configuration`. There you will find a claim with name `token_introspection_endpoint` within the response. Only `confidential clients` are allowed to invoke the new endpoint, where these clients will be usually acting as a resource server and looking for token metadata in order to perform local authorization checks.

**Migrating to 1.7.0.CR1**

**Direct access grants disabled by default for clients**

In order to add more compliance with OpenID Connect specification, we added flags into admin console to Client Settings page, where you can enable/disable various kinds of OpenID Connect/OAuth2 flows (Standard flow, Implicit flow, Direct Access Grants, Service Accounts). As part of this, we have `Direct Access Grants` (corresponds to OAuth2 `Resource Owner Password Credentials Grant`) disabled by default for new clients.

Clients migrated from previous version have `Direct Access Grants` enabled just if they had flag `Direct Grants Only` on. The `Direct Grants Only` flag was removed as if you enable Direct Access Grants and disable both Standard+Implicit flow, you will achieve same effect.

We also added built-in client `admin-cli` to each realm. This client has `Direct Access Grants` enabled. So if you’re using Admin REST API or Keycloak admin-client, you should update your configuration to use `admin-cli` instead of `security-admin-console` as the latter one doesn’t have direct access grants enabled anymore by default.

**Option 'Update Profile On First Login' moved from Identity provider to Review Profile authenticator**

In this version, we added `First Broker Login`, which allows you to specify what exactly should be done when new user is logged through Identity provider (or Social provider), but there is no existing Keycloak user yet linked to the social account. As part of this work, we added option `First Login Flow` to identity providers where you can specify the flow and then you can configure this flow under `Authentication` tab in admin console.

We also removed the option `Update Profile On First Login` from the Identity provider settings and moved it to the configuration of `Review Profile` authenticator. So once you specify which flow should be used for your Identity provider (by default it’s `First Broker Login` flow), you go to `Authentication` tab, select the flow and then you configure the option under `Review Profile` authenticator.

**Element 'form-error-page' in web.xml not supported anymore**

form-error-page in web.xml will no longer work for client adapter authentication errors. You must define an error-page for the various HTTP error codes. See documentation for more details if you want to catch and handle adapter error conditions.

**IdentityProviderMapper changes**

There is no change in the interface itself or method signatures. However there is some change in behavior. We added `First Broker Login` flow in this release and the method `IdentityProviderMapper.importNewUser` is now called after `First Broker Login` flow is finished. So if you want to have any attribute available in `Review Profile` page, you would need to use the method `preprocessFederatedIdentity` instead of `importNewUser` . You can set any attribute by invoke `BrokeredIdentityContext.setUserAttribute` and that will be available on `Review profile` page.

**Migrating to 1.6.0.Final**

**Option that refresh tokens are not reusable anymore**

Old versions of Keycloak allowed reusing refresh tokens multiple times. Keycloak still permits this, but also have an option `Revoke refresh token` to disallow it. Option is in in admin console under token settings. When a refresh token is used to obtain a new access token a new refresh token is also included. When option is enabled, then this new refresh token should be used next time the access token is refreshed. It won’t be possible to reuse old refresh token multiple times.

**Some packages renamed**

We did a bit of restructure and renamed some packages. It is mainly about renaming internal packages of util classes. The most important classes used in your application ( for example AccessToken or KeycloakSecurityContext ) as well as the SPI are still unchanged. However there is slight chance that you will be affected and will need to update imports of your classes. For example if you are using multitenancy and KeycloakConfigResolver, you will be affected as for example class HttpFacade was moved to different package - it is `org.keycloak.adapters.spi.HttpFacade` now.

**Persisting user sessions**

We added support for offline tokens in this release, which means that we are persisting "offline" user sessions into database now. If you are not using offline tokens, nothing will be persisted for you, so you don’t need to care about worse performance for more DB writes. However in all cases, you will need to update `standalone/configuration/keycloak-server.json` and add `userSessionPersister` like this:

```
"userSessionPersister": {
    "provider": "jpa"
},
```

If you want sessions to be persisted in Mongo instead of classic RDBMS, use provider `mongo` instead.

**Migrating to 1.5.0.Final**

**Realm and User cache providers**

Infinispan is now the default and only realm and user cache providers. In non-clustered mode a local Infinispan cache is used. We’ve also removed our custom in-memory cache and the no cache providers. If you have realmCache or userCache set in keycloak-server.json to mem or none please remove these. As Infinispan is the only provider there’s no longer any need for the realmCache and userCache objects so these can be removed.

**Uses Session providers**

Infinispan is now the default and only user session provider. In non-clustered mode a local Infinispan cache is used. We’ve also removed the JPA and Mongo user session providers. If you have userSession set in keycloak-server.json to mem, jpa or mongo please remove it. As Infinispan is the only provider there’s no longer any need for the userSession object so it can be removed.

For anyone that wants to achieve increased durability of user sessions this can be achieved by configuring the user session cache with more than one owner or use a replicated cache. It’s also possible to configure Infinispan to persist caches, although that would have impacts on performance.

**Contact details removed from registration and account management**

In the default theme we have now removed the contact details from the registration page and account management. The admin console now lists all the users attributes, not just contact specific attributes. The admin console also has the ability to add/remove attributes to a user. If you want to add contact details, please refer to the address theme included in the examples.

**Migrating to 1.3.0.Final**

**Direct Grant API always enabled**

In the past Direct Grant API (or Resource Owner Password Credentials) was disabled by default and there was an option on a realm to enable it. The Direct Grant API is now always enabled and the option to enable/disable for a realm is removed.

**Database changed**

There are again few database changes. Remember to backup your database prior to upgrading.

**UserFederationProvider changed**

There are few minor changes in UserFederationProvider interface. You may need to sync your implementation when upgrade to newer version and upgrade few methods, which has changed signature. Changes are really minor, but were needed to improve performance of federation.

**WildFly 9.0.0.Final**

Following on from the distribution changes that was done in the last release the standalone download of Keycloak is now based on WildFly 9.0.0.Final. This also affects the overlay which can only be deployed to WildFly 9.0.0.Final or JBoss EAP 6.4.0.GA. WildFly 8.2.0.Final is no longer supported for the server.

**WildFly, JBoss EAP and JBoss AS7 adapters**

There are now 3 separate adapter downloads for WildFly, JBoss EAP and JBoss AS7:

* eap6
* wf9
* wf8
* as7

Make sure you grab the correct one.

You also need to update standalone.xml as the extension module and subsystem definition has changed. See [Adapter Installation](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/MigrationFromOlderVersions.html#\_jboss\_adapter\_installation) for details.

**Migrating from 1.2.0.Beta1 to 1.2.0.RC1**

**Distribution changes**

Keycloak is now available in 3 downloads: standalone, overlay and demo bundle. The standalone is intended for production and non-JEE developers. Overlay is aimed at adding Keycloak to an existing WildFly 8.2 or EAP 6.4 installation and is mainly for development. Finally we have a demo (or dev) bundle that is aimed at developers getting started with Keycloak. This bundle contains a WildFly server, with Keycloak server and adapter included. It also contains all documentation and examples.

**Database changed**

This release contains again a number of changes to the database. The biggest one is Application and OAuth client merge. Remember to backup your database prior to upgrading.

**Application and OAuth client merge**

Application and OAuth clients are now merged into `Clients`. The UI of admin console is updated and database as well. Your data from database should be automatically updated. The previously set Applications will be converted into Clients with `Consent required` switch off and OAuth Clients will be converted into Clients with this switch on.

**Migrating from 1.1.0.Final to 1.2.0.Beta1**

**Database changed**

This release contains a number of changes to the database. Remember to backup your database prior to upgrading.

**`iss` in access and id tokens**

The value of `iss` claim in access and id tokens have changed from `realm name` to `realm url`. This is required by OpenID Connect specification. If you’re using our adapters there’s no change required, other than if you’ve been using bearer-only without specifying `auth-server-url` you have to add it now. If you’re using another library (or RSATokenVerifier) you need to make the corresponding changes when verifying `iss`.

**OpenID Connect endpoints**

To comply with OpenID Connect specification the authentication and token endpoints have been changed to having a single authentication endpoint and a single token endpoint. As per-spec `response_type` and `grant_type` parameters are used to select the required flow. The old endpoints (`/realms/{realm}/protocols/openid-connect/login`, `/realms/{realm}/protocols/openid-connect/grants/access`, `/realms/{realm}/protocols/openid-connect/refresh`, `/realms/{realm}/protocols/openid-connect/access/codes)` are now deprecated and will be removed in a future version.

**Theme changes**

The layout of themes have changed. The directory hierarchy used to be `type/name` this is now changed to `name/type`. For example a login theme named `sunrise` used to be deployed to `standalone/configuration/themes/login/sunrise`, which is now moved to `standalone/configuration/themes/sunrise/login`. This change was done to make it easier to have group the different types for the same theme into one folder.

If you deployed themes as a JAR in the past you had to create a custom theme loader which required Java code. This has been simplified to only requiring a plain text file (`META-INF/keycloak-themes.json`) to describe the themes included in a JAR. See the [themes](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/MigrationFromOlderVersions.html#\_themes) section in the docs for more information.

**Claims changes**

Previously there was `Claims` tab in admin console for application and OAuth clients. This was used to configure which attributes should go into access token for particular application/client. This was removed and replaced with [Protocol mappers](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/MigrationFromOlderVersions.html#\_mappers), which are more flexible.

You don’t need to care about migration of database from previous version. We did migration scripts for both RDBMS and Mongo, which should ensure that claims configured for particular application/client will be converted into corresponding protocol mappers (Still it’s safer to backup DB before migrating to newer version though). Same applies for exported JSON representation from previous version.

**Social migration to identity brokering**

We refactored social providers SPI and replaced it with [identity brokering SPI](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/MigrationFromOlderVersions.html#\_identity\_broker), which is more flexible. The `Social` tab in admin console is renamed to `Identity Provider` tab.

Again you don’t need to care about migration of database from previous version similarly like for Claims/protocol mappers. Both configuration of social providers and "social links" to your users will be converted to corresponding Identity providers.

Only required action from you would be to change allowed `Redirect URI` in the admin console of particular 3rd party social providers. You can first go to the Keycloak admin console and copy Redirect URI from the page where you configure the identity provider. Then you can simply paste this as allowed Redirect URI to the admin console of 3rd party provider (IE. Facebook admin console).

**Migrating from 1.1.0.Beta2 to 1.1.0.Final**

* WEB-INF/lib +`standalone/configuration/providers`[+providers](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/MigrationFromOlderVersions.html#\_providers)

**Migrating from 1.1.0.Beta1 to 1.1.0.Beta2**

* Adapters are now a separate download. They are not included in appliance and war distribution. We have too many now and the distro is getting bloated.
* org.keycloak.adapters.tomcat7.KeycloakAuthenticatorValve +`org.keycloak.adapters.tomcat.KeycloakAuthenticatorValve`
* JavaScript adapter now has idToken and idTokenParsed properties. If you use idToken to retrieve first name, email, etc. you need to change this to idTokenParsed.
* The as7-eap-subsystem and keycloak-wildfly-subsystem have been merged into one keycloak-subsystem. If you have an existing standalone.xml or domain.xml, you will need edit near the top of the file and change the extension module name to org.keycloak.keycloak-subsystem. For AS7 only, the extension module name is org.keycloak.keycloak-as7-subsystem.
* Server installation is no longer supported on AS7. You can still use AS7 as an application client.

**Migrating from 1.0.x.Final to 1.1.0.Beta1**

* RealmModel JPA and Mongo storage schema has changed
* UserSessionModel JPA and Mongo storage schema has changed as these interfaces have been refactored
* Upgrade your adapters, old adapters are not compatible with Keycloak 1.1. We interpreted JSON Web Token and OIDC ID Token specification incorrectly. 'aud' claim must be the client id, we were storing the realm name in there and validating it.

**Migrating from 1.0 RC-1 to RC-2**

* A lot of info level logging has been changed to debug. Also, a realm no longer has the jboss-logging audit listener by default. If you want log output when users login, logout, change passwords, etc. enable the jboss-logging audit listener through the admin console.

**Migrating from 1.0 Beta 4 to RC-1**

* logout REST API has been refactored. The GET request on the logout URI does not take a session\_state parameter anymore. You must be logged in in order to log out the session. You can also POST to the logout REST URI. This action requires a valid refresh token to perform the logout. The signature is the same as refresh token minus the grant type form parameter. See documentation for details.

**Migrating from 1.0 Beta 1 to Beta 4**

* LDAP/AD configuration is changed. It is no longer under the "Settings" page. It is now under Users→Federation. Add Provider will show you an "ldap" option.
* Authentication SPI has been removed and rewritten. The new SPI is UserFederationProvider and is more flexible.
* ssl-not-required +`ssl-required` +`all` +`external` +`none`
* DB Schema has changed again.
* Created applications now have a full scope by default. This means that you don’t have to configure the scope of an application if you don’t want to.
* Format of JSON file for importing realm data was changed. Now role mappings is available under the JSON record of particular user.

**Migrating from 1.0 Alpha 4 to Beta 1**

* DB Schema has changed. We have added export of the database to Beta 1, but not the ability to import the database from older versions. This will be supported in future releases.
* For all clients except bearer-only applications, you must specify at least one redirect URI. Keycloak will not allow you to log in unless you have specified a valid redirect URI for that application.
* Direct Grant API +`ON`
* standalone/configuration/keycloak-server.json
* JavaScript adapter
* Session Timeout

**Migrating from 1.0 Alpha 2 to Alpha 3**

* SkeletonKeyToken, SkeletonKeyScope, SkeletonKeyPrincipal, and SkeletonKeySession have been renamed to: AccessToken, AccessScope, KeycloakPrincipal, and KeycloakAuthenticatedSession respectively.
* ServleOAuthClient.getBearerToken() method signature has changed. It now returns an AccessTokenResponse so that you can obtain a refresh token too.
* Adapters now check the access token expiration with every request. If the token is expired, they will attempt to invoke a refresh on the auth server using a saved refresh token.
* Subject in AccessToken has been changed to the User ID.

**Migrating from 1.0 Alpha 1 to Alpha 2**

* DB Schema has changed. We don’t have any data migration utilities yet as of Alpha 2.
* JBoss and Wildfly adapters are now installed via a JBoss/Wildfly subsystem. Please review the adapter installation documentation. Edits to standalone.xml are now required.
* There is a new credential type "secret". Unlike other credential types, it is stored in plain text in the database and can be viewed in the admin console.
* There is no longer required Application or OAuth Client credentials. These client types are now hard coded to use the "secret" credential type.
* Because of the "secret" credential change to Application and OAuth Client, you’ll have to update your keycloak.json configuration files and regenarate a secret within the Application or OAuth Client credentials tab in the administration console.
