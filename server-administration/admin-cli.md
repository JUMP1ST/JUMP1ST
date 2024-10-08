# Admin CLI

In previous chapters we have described how to use the Keycloak Admin Console to perform administrative tasks. All those tasks can also be performed from command line by using Admin CLI command line tool.

#### Installing Admin CLI <a href="#installing_admin_cli" id="installing_admin_cli"></a>

Admin CLI is packaged inside Keycloak Server distribution. You can find execution scripts inside `bin` directory.

The Linux script is called `kcadm.sh`, and the one for Windows is called `kcadm.bat`.

In order to setup the client to be used from any location on the filesystem you may want to add Keycloak server directory to your PATH.

On Linux:

```
$ export PATH=$PATH:$KEYCLOAK_HOME/bin
$ kcadm.sh
```

On Windows:

```
c:\> set PATH=%PATH%;%KEYCLOAK_HOME%\bin
c:\> kcadm
```

| Note | To avoid unnecessary repetition the rest of this document will only give Windows examples in places where difference in command line is more than just in `kcadm` command name. |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

#### Using Admin CLI <a href="#using_admin_cli" id="using_admin_cli"></a>

Usually a user will first start an authenticated session by providing credentials, then perform some CRUD operations.

For example on Linux:

```
$ kcadm.sh config credentials --server http://localhost:8080/auth --realm demo --user admin --client admin
$ kcadm.sh create realms -s realm=demorealm -s enabled=true -o
$ CID=$(kcadm.sh create clients -r demorealm -s clientId=my_client -s 'redirectUris=["http://localhost:8980/myapp/*"]' -i)
$ kcadm.sh get clients/$CID/installation/providers/keycloak-oidc-keycloak-json
```

Or on Windows:

```
c:\> kcadm config credentials --server http://localhost:8080/auth --realm demo --user admin --client admin
c:\> kcadm create realms -s realm=demorealm -s enabled=true -o
c:\> kcadm create clients -r demorealm -s clientId=my_client -s "redirectUris=[\"http://localhost:8980/myapp/*\"]" -i > clientid.txt
c:\> set /p CID=<clientid.txt
c:\> kcadm get clients/%CID%/installation/providers/keycloak-oidc-keycloak-json
```

In a production environment Keycloak has to be accessed with `https:` to avoid exposing tokens to network sniffers. If server’s certificate is not issued by one of the trusted CAs that are included in Java’s default certificate truststore, then you will need to prepare a truststore.jks file, and instruct `Admin CLI` to use it.

For example on Linux:

```
$ kcadm.sh config truststore --trustpass $PASSWORD ~/.keycloak/truststore.jks
```

Or on Windows:

```
c:\> kcadm config truststore --trustpass %PASSWORD% %HOMEPATH%\.keycloak\truststore.jks
```

#### Authenticating <a href="#authenticating" id="authenticating"></a>

Admin CLI works by making HTTP requests to Admin REST endpoints. Access to them is protected and requires authentication.

When logging in with `Admin CLI` you specify a server endpoint url, and a realm. Then you specify a username, or alternatively you can only specify a client id, which will result in special service account being used. In the first case, a password for the specified user has to be used at login. In the latter case there is no user password - only client secret or a `Signed JWT` is used.

The account that logs in needs to have proper permissions in order to be able to invoke Admin REST API operations. Specifically - `realm-admin` role of `realm-management` client is required for user to administer the realm within which the user is defined.

There are two primary mechanisms to authenticate. One is by using `kcadm config credentials` to start an authenticated session:

```
$ kcadm.sh config credentials --server http://localhost:8080/auth --realm master --user admin --password admin
```

This approach maintains an authenticated session between `kcadm` command invocations by saving the obtained access token, and associated refresh token, possibly other secrets as well in a private configuration file. By default this file is called `kcadm.config` and is located under user’s home directory - it’s full pathname is `$HOME/.keycloak/kcadm.config` (on Windows it’s `%HOMEPATH%\.keycloak\kcadm.config`). The file can be named something else by using `-c, --config` option.

See [next chapter](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/fake/.html#\_working\_with\_alternative\_configurations) for more info on configuration file.

Another approach is to authenticate with each command invocation for the duration of that invocation only. This approach results in more load on the server, and more time spent with round-trips obtaining tokens, but has a benefit of not needing to save any tokens between invocations, thus nothing is saved to disk.

For example, when performing an operation we specify all the information required for authentication:

```
$ kcadm.sh get realms --no-config --server http://localhost:8080/auth --realm master --user admin --password admin
```

See built-in help for more information on using `Admin CLI`.

For example:

```
$ kcadm.sh help
```

See `kcadm.sh config credentials --help` for more information about starting an authenticated session.

#### Working with alternative configurations <a href="#working_with_alternative_configurations" id="working_with_alternative_configurations"></a>

By default, `Admin CLI` automatically maintains a configuration file at a default location - `.keycloak/kcadm.config` under user’s home directory.

You can use `--config` option at any time to point to a different file / location. This way you can maintain multiple authenticated sessions in parallel. It is safest to perform operations tied to a single config file from a single thread.

Make sure to not make a config file visible to other users on the system as it contains access tokens, and secrets that should be kept private.

You may want to avoid storing any secrets at all inside a config file for the price of less convenience and having to do more token requests. In that case you can use `--no-config` option with all your commands. You will have to specify all authentication info with each `kcadm` invocation.

#### Basic operations, and resource URIs <a href="#basic_operations_and_resource_uris" id="basic_operations_and_resource_uris"></a>

Admin CLI allows you to perform CRUD operations against Admin REST API endpoints in quite a generic way, with additional commands that simplify performing certain actions.

Main usage pattern is the following:

```
$ kcadm.sh create ENDPOINT [ARGUMENTS]
$ kcadm.sh get ENDPOINT [ARGUMENTS]
$ kcadm.sh update ENDPOINT [ARGUMENTS]
$ kcadm.sh delete ENDPOINT [ARGUMENTS]
```

Where operations `create`, `get`, `update`, and `delete` are mapped to HTTP verbs POST, GET, PUT, and DELETE, respectively. ENDPOINT is a target resource URI, and can either be absolute - starting with 'http:' or 'https:', or relative - used to compose an absolute URL of the following format:

```
SERVER_URI/admin/realms/REALM/ENDPOINT
```

For example, if the server we authenticate against is `http://localhost:8080/auth`, and realm is `master`, then using `users` as ENDPOINT will result in the following resource URL: `http://localhost:8080/auth/admin/realms/master/users`.

If we set ENDPOINT to `clients` the effective resource URI would be: `http://localhost:8080/auth/admin/realms/master/clients`.

There is `realms` endpoint which is treated slightly differently since it is the container for realms. It resolves simply to:

```
SERVER_URI/admin/realms
```

There is also `serverinfo` which is treated the same way since it is independent of realms.

When authenticating as a user with realm-admin powers you may need to perform operations on multiple different realms. In that case you can specify `-r, --target-realm` option to tell explicitly which realm the operation should be executed against. Instead of using REALM as specified via `--realm` option of `kcadm.sh config credentials`, the TARGET\_REALM will be used:

```
SERVER_URI/admin/realms/TARGET_REALM/ENDPOINT
```

For example:

```
$ kcadm.sh config credentials --server http://localhost:8080/auth --realm master --user admin --password admin
$ kcadm.sh create users -s username=testuser -s enabled=true -r demorealm
```

In this example we first start a session authenticated as `admin` user in `master` realm. Then we perform a POST call against the following resource URL:

```
http://localhost:8080/auth/admin/realms/demorealm/users
```

#### Realm operations <a href="#realm_operations" id="realm_operations"></a>

Creating a new realm

A new realm can be created by specifying individual attributes on command line. They will be converted into a JSON document and sent to the server:

```
$ kcadm.sh create realms -s realm=demorealm -s enabled=true
```

Realm is not enabled by default. By enabling it, it can be used for authentication immediately.

A description for a new object can be in JSON format as well:

```
$ kcadm.sh create realms -f demorealm.json
```

JSON document with realm attributes can be sent directly from file or piped to standard input.

For example on Linux:

```
$ kcadm.sh create realms -f - << EOF
{ "realm": "demorealm", "enabled": true }
EOF
```

Or on Windows:

```
c:\> echo { "realm": "demorealm", "enabled": true } | kcadm create realms -f -
```

Listing existing realms

The following will return a list of all the realms:

```
$ kcadm.sh get realms
```

Note, that the list of realms returned is additionally filtered on the server to only return realms the user has permissions for.

Often that is too much information as we may only be interested in realm name, or - for example - if it is enabled or not. You can specify the attributes to return by using `--fields` option:

```
$ kcadm.sh get realms --fields realm,enabled
```

You may even display the result as comma separated values:

```
$ kcadm.sh get realms --fields realm --format csv --noquotes
```

Getting a specific realm

As is common for REST web services, in order to get an individual item of a collection, append an id to collection URI:

```
$ kcadm.sh get realms/master
```

Updating a realm

There are several options when updating any resource. You can first get current state of resource, and save it into a file, then edit that file, and send it to server for update. For example:

```
$ kcadm.sh get realms/demorealm > demorealm.json
$ vi demorealm.json
$ kcadm.sh update realms/demorealm -f demorealm.json
```

This way the resource on the server will be updated with all the attributes in the sent JSON document.

Another option is to perform the update on-the-fly using `-s, --set` options to set new values:

```
$ kcadm.sh update realms/demorealm -s enabled=false
```

That would only update `enabled` attribute to `false`.

Deleting a realm

It’s very simple to delete a realm:

```
$ kcadm.sh delete realms/demorealm
```

Turning on all login page options for the realm

Set the attributes controlling specific capabilities to `true`.

For example:

```
$ kcadm.sh update realms/demorealm -s registrationAllowed=true -s registrationEmailAsUsername=true -s rememberMe=true -s verifyEmail=true -s resetPasswordAllowed=true -s editUsernameAllowed=true
```

Listing the realm keys

It’s very simple to list the realm keys for a specific realm:

```
$ kcadm.sh get keys -r demorealm
```

Generating new realm keys

To add a new RSA generated keypair, first get `id` of the target realm. For example, to get `id` for a realm whose `realm` attribute is 'demorealm':

```
$ kcadm.sh get realms/demorealm --fields id --format csv --noquotes
```

Then add a new key provider with higher priority than any of the existing providers as revealed by `kcadm.sh get keys -r demorealm`:

For example on Linux:

```
$ kcadm.sh create components -r demorealm -s name=rsa-generated -s providerId=rsa-generated -s providerType=org.keycloak.keys.KeyProvider -s parentId=959844c1-d149-41d7-8359-6aa527fca0b0 -s 'config.priority=["101"]' -s 'config.enabled=["true"]' -s 'config.active=["true"]' -s 'config.keySize=["2048"]'
```

Or on Windows:

```
c:\> kcadm create components -r demorealm -s name=rsa-generated -s providerId=rsa-generated -s providerType=org.keycloak.keys.KeyProvider -s parentId=959844c1-d149-41d7-8359-6aa527fca0b0 -s "config.priority=[\"101\"]" -s "config.enabled=[\"true\"]" -s "config.active=[\"true\"]" -s "config.keySize=[\"2048\"]"
```

Attribute `parentId` should be set to the value of target realm’s `id`.

The newly added key should now become the active key as revealed by `kcadm.sh get keys -r demorealm`.

Adding new realm keys from Java Key Store file

To add a new keypair already prepared as a JKS file on the server, add a new key provider as follows:

For exmple on Linux:

```
$ kcadm.sh create components -r demorealm -s name=java-keystore -s providerId=java-keystore -s providerType=org.keycloak.keys.KeyProvider -s parentId=959844c1-d149-41d7-8359-6aa527fca0b0 -s 'config.priority=["101"]' -s 'config.enabled=["true"]' -s 'config.active=["true"]' -s 'config.keystore=["/opt/keycloak/keystore.jks"]' -s 'config.keystorePassword=["secret"]' -s 'config.keyPassword=["secret"]' -s 'config.alias=["localhost"]'
```

Or on Windows:

```
c:\> kcadm create components -r demorealm -s name=java-keystore -s providerId=java-keystore -s providerType=org.keycloak.keys.KeyProvider -s parentId=959844c1-d149-41d7-8359-6aa527fca0b0 -s "config.priority=[\"101\"]" -s "config.enabled=[\"true\"]" -s "config.active=[\"true\"]" -s "config.keystore=[\"/opt/keycloak/keystore.jks\"]" -s "config.keystorePassword=[\"secret\"]" -s "config.keyPassword=[\"secret\"]" -s "config.alias=[\"localhost\"]"
```

And change attribute values for `keystore`, `keystorePassword`, `keyPassword`, and `alias` to match your specific keystore.

Attribute `parentId` should be set to the value of target realm’s `id`.

Making key passive or disabling it

Identify the key you wish to make passive:

```
$ kcadm.sh get keys -r demorealm
```

Use `providerId` attribute of the key to construct an endpoint uri - `components/PROVIDER_ID`.

Then perform an `update`. For example on Linux:

```
$ kcadm.sh update components/PROVIDER_ID -r demorealm -s 'config.active=["false"]'
```

Or on Windows:

```
c:\> kcadm update components/PROVIDER_ID -r demorealm -s "config.active=[\"false\"]"
```

Analogously, other key attributes can be updated.

To disable the key set new `enabled` value, for example: `'config.enabled=["false"]'`

To change key’s priority set new `priority` value, for example: `'config.priority=["110"]'`

Deleting an old key

Make sure that the key you are deleting has been passive for some time, and then disabled for some time in order to prevent any existing tokens held by applications and users from abruptly failing to work.

Identify the key you wish to make passive:

```
$ kcadm.sh get keys -r demorealm
```

Use the `providerId` of that key to perform a delete. For example:

```
$ kcadm.sh delete components/PROVIDER_ID -r demorealm
```

Configuring event logging for a realm

Use `update` against `events/config` endpoint.

Attribute 'eventsListeners' sets the list of EventListenerProviderFactory 'id’s specifying all the event listeners receiving events. Separately from that there are attributes that control a built-in event storage which allows querying of past events via Admin REST API. There is separate control over logging of service calls - 'eventsEnabled', and auditing events triggered during Admin Console or Admin REST API - 'adminEventsEnabled'. You may want to limit the time when old events expire so that your database doesn’t get filled up - 'eventsExpiration' is set to time-to-live expressed in seconds.

For example, this is how you set a built-in event listener that will receive all the events and log them through jboss-logging (error events are logged as `WARN`, others as `DEBUG`, using a logger called `org.keycloak.events`):

On Linux:

```
$ kcadm.sh update events/config -r demorealm -s 'eventsListeners=["jboss-logging"]'
```

Or on Windows:

```
c:\> kcadm update events/config -r demorealm -s "eventsListeners=[\"jboss-logging\"]"
```

This is how you turn on storage of all available ERROR events - not auditing events - for 2 days so they can be retrieved via Admin REST:

On Linux:

```
$ kcadm.sh update events/config -r demorealm -s eventsEnabled=true -s 'enabledEventTypes=["LOGIN_ERROR","REGISTER_ERROR","LOGOUT_ERROR","CODE_TO_TOKEN_ERROR","CLIENT_LOGIN_ERROR","FEDERATED_IDENTITY_LINK_ERROR","REMOVE_FEDERATED_IDENTITY_ERROR","UPDATE_EMAIL_ERROR","UPDATE_PROFILE_ERROR","UPDATE_PASSWORD_ERROR","UPDATE_TOTP_ERROR","VERIFY_EMAIL_ERROR","REMOVE_TOTP_ERROR","SEND_VERIFY_EMAIL_ERROR","SEND_RESET_PASSWORD_ERROR","SEND_IDENTITY_PROVIDER_LINK_ERROR","RESET_PASSWORD_ERROR","IDENTITY_PROVIDER_FIRST_LOGIN_ERROR","IDENTITY_PROVIDER_POST_LOGIN_ERROR","CUSTOM_REQUIRED_ACTION_ERROR","EXECUTE_ACTIONS_ERROR","CLIENT_REGISTER_ERROR","CLIENT_UPDATE_ERROR","CLIENT_DELETE_ERROR"]' -s eventsExpiration=172800
```

Or on Windows:

```
c:\> kcadm update events/config -r demorealm -s eventsEnabled=true -s "enabledEventTypes=[\"LOGIN_ERROR\",\"REGISTER_ERROR\",\"LOGOUT_ERROR\",\"CODE_TO_TOKEN_ERROR\",\"CLIENT_LOGIN_ERROR\",\"FEDERATED_IDENTITY_LINK_ERROR\",\"REMOVE_FEDERATED_IDENTITY_ERROR\",\"UPDATE_EMAIL_ERROR\",\"UPDATE_PROFILE_ERROR\",\"UPDATE_PASSWORD_ERROR\",\"UPDATE_TOTP_ERROR\",\"VERIFY_EMAIL_ERROR\",\"REMOVE_TOTP_ERROR\",\"SEND_VERIFY_EMAIL_ERROR\",\"SEND_RESET_PASSWORD_ERROR\",\"SEND_IDENTITY_PROVIDER_LINK_ERROR\",\"RESET_PASSWORD_ERROR\",\"IDENTITY_PROVIDER_FIRST_LOGIN_ERROR\",\"IDENTITY_PROVIDER_POST_LOGIN_ERROR\",\"CUSTOM_REQUIRED_ACTION_ERROR\",\"EXECUTE_ACTIONS_ERROR\",\"CLIENT_REGISTER_ERROR\",\"CLIENT_UPDATE_ERROR\",\"CLIENT_DELETE_ERROR\"]" -s eventsExpiration=172800
```

This is how you reset stored event types to `all available event types` - setting to empty list is the same as enumerating all:

```
$ kcadm.sh update events/config -r demorealm -s enabledEventTypes=[]
```

And this is how you turn on auditing events:

```
$ kcadm.sh update events/config -r demorealm -s adminEventsEnabled=true -s adminEventsDetailsEnabled=true
```

Here is how you get the last 100 events - they are ordered from newest to oldest:

```
$ kcadm.sh get events --offset 0 --limit 100
```

Here is how you delete all saved events:

```
$ kcadm delete events
```

Flushing the caches

Use `create` operation, and one of the following endpoints: `clear-realm-cache`, `clear-user-cache`, `clear-keys-cache`.

Set `realm` to the same value as target realm.

For example:

```
$ kcadm.sh create clear-realm-cache -r demorealm -s realm=demorealm
```

```
$ kcadm.sh create clear-user-cache -r demorealm -s realm=demorealm
```

```
$ kcadm.sh create clear-keys-cache -r demorealm -s realm=demorealm
```

#### Role operations <a href="#role_operations" id="role_operations"></a>

Creating a realm role

To create a realm role use `roles` endpoint:

```
$ kcadm.sh create roles -r demorealm -s name=user -s 'description=Regular user with limited set of permissions'
```

Creating a client role

To create a client role identify the client first - use `get` to list available clients:

```
$ kcadm.sh get clients -r demorealm --fields id,clientId
```

Then create a new role by using client’s `id` attribute to construct an endpoint uri - `clients/ID/roles`.

For example:

```
$ kcadm.sh create clients/a95b6af3-0bdc-4878-ae2e-6d61a4eca9a0/roles -r demorealm -s name=editor -s 'description=Editor can edit, and publish any article'
```

Listing realm roles

To list existing realm roles use `get` command:

```
$ kcadm.sh get roles -r demorealm
```

You can also use `get-roles` command:

```
$ kcadm.sh get-roles -r demorealm
```

Listing client roles

Use special `get-roles` command, passing it either `clientId` (via `--cclientid` option) or `id` (via `--cid` option) to identify the client, and list defined roles:

For example:

```
$ kcadm.sh get-roles -r demorealm --cclientid realm-management
```

Getting a specific realm role

Use `get` command, and role `name` to construct an endpoint uri for a specific realm role - `roles/ROLE_NAME`

For example:

```
$ kcadm.sh get roles/user -r demorealm
```

Where `user` is the name of existing role.

Alternatively, use special `get-roles` command, passing it role `name` (via `--rolename` option) or `id` (via `--roleid` option).

For example:

```
$ kcadm.sh get-roles -r demorealm --rolename user
```

Getting a specific client role

Use special `get-roles` command, passing it either `clientId` (via `--cclientid` option) or `id` (via `--cid` option) to identify the client, and passing it either role `name` (via `--rolename` option) or 'id' (via --roleid) to identify a specific client role:

For example:

```
$ kcadm.sh get-roles -r demorealm --cclientid realm-management --rolename manage-clients
```

Updating a realm role

Use `update` operation with the same endpoint uri as for getting a specific realm role. For example:

```
$ kcadm.sh update roles/user -r demorealm -s 'description=Role representing a regular user'
```

Updating a client role

Use `update` operation with the same endpoint uri as for getting a specific client role. For example:

```
$ kcadm.sh update clients/a95b6af3-0bdc-4878-ae2e-6d61a4eca9a0/roles/editor -r demorealm -s 'description=User that can edit, and publish articles'
```

Deleting a realm role

Use `delete` operation with the same endpoint uri as for getting a specific realm role. For example:

```
$ kcadm.sh delete roles/user -r demorealm
```

Deleting a client role

Use `delete` operation with the same endpoint uri as for getting a specific client role. For example:

```
$ kcadm.sh delete clients/a95b6af3-0bdc-4878-ae2e-6d61a4eca9a0/roles/editor -r demorealm
```

Listing assigned, available and effective realm roles for a composite role

There is a dedicated `get-roles` command to simplify listing of both realm and client roles. It is an extension of `get` command thus it behaves like `get` command with additional semantics for listing roles.

To list **assigned** realm roles for the composite role you can specify the target composite role by either `name` (via --rname option) or `id` (via --rid option).

For example:

```
$ kcadm.sh get-roles -r demorealm --rname testrole
```

To list **effective** realm roles, use additional `--effective` option.

For example:

```
$ kcadm.sh get-roles -r demorealm --rname testrole --effective
```

To list realm roles that can still be added to the composite role, use `--available` option instead.

For example:

```
$ kcadm.sh get-roles -r demorealm --rname testrole --available
```

Listing assigned, available, and effective client roles for a composite role

You can again use `get-roles` command to simplify listing of roles.

To list **assigned** client roles for the composite role you can specify the target composite role by either `name` (via --rname option) or `id` (via --rid option), and client by either `clientId` (via --cclientid option) or `id` (via --cid option).

For example:

```
$ kcadm.sh get-roles -r demorealm --rname testrole --cclientid realm-management
```

To list **effective** realm roles, use additional `--effective` option.

For example:

```
$ kcadm.sh get-roles -r demorealm --rname testrole --cclientid realm-management --effective
```

To list realm roles that can still be added to the target composite role, use `--available` option instead.

For example:

```
$ kcadm.sh get-roles -r demorealm --rname testrole --cclientid realm-management --available
```

Adding realm roles to a composite role

There is a dedicated `add-roles` command that can be used for adding both realm roles and client roles.

For example, to add 'user' role to composite role 'testrole' :

```
$ kcadm.sh add-roles --rname testrole --rolename user -r demorealm
```

Removing realm roles from a composite role

There is a dedicated `remove-roles` command that can be used to remove both realm roles and client roles.

For example, to remove 'user' role from target composite role 'testrole':

```
$ kcadm.sh remove-roles --rname testrole --rolename user -r demorealm
```

Adding client roles to a composite role

There is a dedicated `add-roles` operation that can be used for adding both realm roles and client roles.

For example, to add to `testrole` composite role two roles defined on client `realm-management` - `create-client` role and `view-users` role:

```
$ kcadm.sh add-roles -r demorealm --rname testrole --cclientid realm-management --rolename create-client --rolename view-users
```

Removing client roles from a composite role

There is a dedicated `remove-roles` operation that can be used for removing both realm roles and client roles.

For example, to remove from `testrole` composite role two roles defined on client `realm management` - `create-client` role and `view-users` role:

```
$ kcadm.sh remove-roles -r demorealm --rname testrole --cclientid realm-management --rolename create-client --rolename view-users
```

#### Client operations <a href="#client_operations" id="client_operations"></a>

Creating a client

A new client can be created by using `create` command against `clients` endpoint. For example:

```
$ kcadm.sh create clients -r demorealm -s clientId=myapp -s enabled=true
```

Listing clients

It’s very easy to list existing clients. For example:

```
$ kcadm.sh get clients -r demorealm --fields id,clientId
```

Here we filter the output to only list `id`, and `clientId` attributes.

Getting a specific client

Use client’s `id` to construct an endpoint uri targeting specific client - `clients/ID`. For example:

```
$ kcadm.sh get clients/c7b8547f-e748-4333-95d0-410b76b3f4a3 -r demorealm
```

Getting adapter configuration file (keycloak.json) for specific client

Use client’s `id` to construct an endpoint uri targeting specific client - `clients/ID/installation/providers/keycloak-oidc-keycloak-json`.

For example:

```
$ kcadm.sh get clients/c7b8547f-e748-4333-95d0-410b76b3f4a3/installation/providers/keycloak-oidc-keycloak-json -r demorealm
```

Getting Wildfly subsystem adapter configuration for specific client

Use client’s `id` to construct an endpoint uri targeting specific client - `clients/ID/installation/providers/keycloak-oidc-jboss-subsystem`.

For example:

```
$ kcadm.sh get clients/c7b8547f-e748-4333-95d0-410b76b3f4a3/installation/providers/keycloak-oidc-jboss-subsystem -r demorealm
```

Updating a client

Use `update` operation with the same endpoint uri as for getting a specific client. For example on Linux:

```
$ kcadm.sh update clients/c7b8547f-e748-4333-95d0-410b76b3f4a3 -r demorealm -s enabled=false -s publicClient=true -s 'redirectUris=["http://localhost:8080/myapp/*"]' -s baseUrl=http://localhost:8080/myapp -s adminUrl=http://localhost:8080/myapp
```

Or on Windows:

```
c:\> kcadm update clients/c7b8547f-e748-4333-95d0-410b76b3f4a3 -r demorealm -s enabled=false -s publicClient=true -s "redirectUris=[\"http://localhost:8080/myapp/*\"]" -s baseUrl=http://localhost:8080/myapp -s adminUrl=http://localhost:8080/myapp
```

Deleting a client

Use `delete` operation with the same endpoint uri as for getting a specific client. For example:

```
$ kcadm.sh delete clients/c7b8547f-e748-4333-95d0-410b76b3f4a3 -r demorealm
```

#### User operations <a href="#user_operations" id="user_operations"></a>

Creating a user

A new user can be created using the `create` command against the `users` endpoint. For example:

```
$ kcadm.sh create users -r demorealm -s username=testuser -s enabled=true
```

Listing users

Use `users` endpoint to list users. Number of users may be large, and you may want to limit how many are returned:

```
$ kcadm.sh get users -r demorealm --offset 0 --limit 1000
```

It’s also possible to filter users by `username`, `firstName`, `lastName`, or `email`. For example:

```
$ kcadm.sh get users -r demorealm -q email=google.com
$ kcadm.sh get users -r demorealm -q username=testuser
```

Note that filtering doesn’t use exact matching. For example, the above would match the value of `username` attribute against '\*testuser\*' pattern.

You can also filter across multiple attributes by specifying multiple `-q` options, which would return only users that match condition for all the attributes.

Getting a specific user

Use user `id` to compose an endpoint uri matching a specific user - `users/USER_ID`.

For example:

```
$ kcadm.sh get users/0ba7a3fd-6fd8-48cd-a60b-2e8fd82d56e2 -r demorealm
```

Updating a user

Use `update` operation with the same endpoint uri as for getting a specific user. For example on Linux:

```
$ kcadm.sh update users/0ba7a3fd-6fd8-48cd-a60b-2e8fd82d56e2 -r demorealm -s 'requiredActions=["VERIFY_EMAIL","UPDATE_PROFILE","CONFIGURE_TOTP","UPDATE_PASSWORD"]'
```

Or on Windows:

```
c:\> kcadm update users/0ba7a3fd-6fd8-48cd-a60b-2e8fd82d56e2 -r demorealm -s "requiredActions=[\"VERIFY_EMAIL\",\"UPDATE_PROFILE\",\"CONFIGURE_TOTP\",\"UPDATE_PASSWORD\"]"
```

Deleting a user

Use `delete` operation with the same endpoint uri as for getting a specific user. For example:

```
$ kcadm.sh delete users/0ba7a3fd-6fd8-48cd-a60b-2e8fd82d56e2 -r demorealm
```

Resetting user’s password

There is a dedicated `set-password` command specifically to reset user’s password. For example:

```
$ kcadm.sh set-password -r demorealm --username testuser --password NEWPASSWORD --temporary
```

That will set a temporary password for the user, which they will have to change the next time they login.

You can use `--userid` if you want to specify the user by using `id` attribute.

The same can be achieved using the `update` operation against an endpoint constructed from one for getting a specific user - `users/USER_ID/reset-password`.

For example:

```
$ kcadm.sh update users/0ba7a3fd-6fd8-48cd-a60b-2e8fd82d56e2/reset-password -r demorealm -s type=password -s value=NEWPASSWORD -s temporary=true -n
```

The last parameter (`-n`) forces a so called 'no-merge' update which performs a PUT only, without first doing a GET to retrieve current state of the resource. In this case it is necessary since `reset-password` endpoint doesn’t support GET.

Listing assigned, available, and effective realm roles for a user

There is a dedicated `get-roles` command to simplify listing of both realm and client roles. It is an extension of `get` command thus it behaves like `get` command with additional semantics for listing roles.

To list **assigned** realm roles for the user you can specify the target user by either `username` or `id`.

For example:

```
$ kcadm.sh get-roles -r demorealm --uusername testuser
```

To list **effective** realm roles, use additional `--effective` option.

For example:

```
$ kcadm.sh get-roles -r demorealm --uusername testuser --effective
```

To list realm roles that can still be added to the user, use `--available` option instead.

For example:

```
$ kcadm.sh get-roles -r demorealm --uusername testuser --available
```

Listing assigned, available, and effective client roles for a user

You can again use `get-roles` command to simplify listing of roles.

To list **assigned** client roles for the user you can specify the target user by either `username` (via --uusername option) or `id` (via --uid option), and client by either `clientId` (via --cclientid option) or `id` (via --cid option).

For example:

```
$ kcadm.sh get-roles -r demorealm --uusername testuser --cclientid realm-management
```

To list **effective** realm roles, use additional `--effective` option.

For example:

```
$ kcadm.sh get-roles -r demorealm --uusername testuser --cclientid realm-management --effective
```

To list realm roles that can still be added to the user, use `--available` option instead.

For example:

```
$ kcadm.sh get-roles -r demorealm --uusername testuser --cclientid realm-management --available
```

Adding realm roles to a user

There is a dedicated `add-roles` command that can be used for adding both realm roles and client roles.

For example, to add 'user' role to user 'testuser' :

```
$ kcadm.sh add-roles --username testuser --rolename user -r demorealm
```

Removing realm roles from a user

There is a dedicated `remove-roles` command that can be used to remove both realm roles and client roles.

For example, to remove 'user' role from user 'testuser':

```
$ kcadm.sh remove-roles --username testuser --rolename user -r demorealm
```

Adding client roles to a user

There is a dedicated `add-roles` operation that can be used for adding both realm roles and client roles.

For example, to add to user `testuser` two roles defined on client `realm management` - `create-client` role and `view-users` role:

```
$ kcadm.sh add-roles -r demorealm --uusername testuser --cclientid realm-management --rolename create-client --rolename view-users
```

Removing client roles from a user

There is a dedicated `remove-roles` operation that can be used for removing both realm roles and client roles.

For example, to remove from user `testuser` two roles defined on client `realm management` - `create-client` role and `view-users` role:

```
$ kcadm.sh remove-roles -r demorealm --uusername testuser --cclientid realm-management --rolename create-client --rolename view-users
```

Listing user’s sessions

First identify user’s `id` then use it to compose an endpoint uri - `users/ID/sessions`.

Now use `get` to retrieve a list of user’s sessions.

For example:

```
$kcadm get users/6da5ab89-3397-4205-afaa-e201ff638f9e/sessions
```

Logging out user from specific session

To invalidate a session you only need session’s `id`. You can get it by listing user’s sessions.

Use session’s `id` to compose an endpoint uri - `sessions/ID`.

The use `delete` to invalidate it. For example:

```
$ kcadm.sh delete sessions/d0eaa7cc-8c5d-489d-811a-69d3c4ec84d1
```

Logging out user from all sessions

You need user’s `id` to construct an endpoint uri - `users/ID/logout`.

Use 'create' to send logout-from-all-sessions request:

```
$ kcadm.sh create users/6da5ab89-3397-4205-afaa-e201ff638f9e/logout -r demorealm -s realm=demorealm -s user=6da5ab89-3397-4205-afaa-e201ff638f9e
```

#### Group operations <a href="#group_operations" id="group_operations"></a>

Creating a group

Use `create` operation, and `groups` endpoint to create a new group:

```
$ kcadm.sh create groups -r demorealm -s name=Group
```

Listing groups

Use `get` operation, and `groups` endpoint to list groups:

```
$ kcadm.sh get groups -r demorealm
```

Getting a specific group

Use group’s `id` to construct an endpoint uri - groups/GROUP\_ID:

For example:

```
$ kcadm.sh get groups/51204821-0580-46db-8f2d-27106c6b5ded -r demorealm
```

Updating a group

Use `update` operation with the same endpoint uri as for getting a specific group. For example:

```
$ kcadm.sh update groups/51204821-0580-46db-8f2d-27106c6b5ded -s 'attributes.email=[""]' -r demorealm
```

Deleting a group

Use `delete` operation with the same endpoint uri as for getting a specific group. For example:

```
$ kcadm.sh delete groups/51204821-0580-46db-8f2d-27106c6b5ded -r demorealm
```

Creating a sub-group

Find 'id' of the parent group - by listing groups for example. Use that `id` to construct an endpoint uri - groups/GROUP\_ID/children:

For example:

```
$ kcadm.sh create groups/51204821-0580-46db-8f2d-27106c6b5ded/children -r demorealm -s name=SubGroup
```

Moving a group under another group

Find 'id' of existing parent group, and of existing child group. Use parent group’s `id` to construct and endpoint uri - groups/PARENT\_GROUP\_ID/children.

Make 'create' operation against this endpoint, and pass child group `id` as JSON body. For example:

```
$ kcadm.sh create groups/51204821-0580-46db-8f2d-27106c6b5ded/children -r demorealm -s id=08d410c6-d585-4059-bb07-54dcb92c5094
```

Get groups for specific user

To get user’s membership in groups, use user’s `id` to compose a resource URI - `users/USER_ID/groups`

For example:

```
$ kcadm.sh get users/b544f379-5fc4-49e5-8a8d-5cfb71f46f53/groups -r demorealm
```

Adding user to a group

To join user to a group use `update` operation against a resource uri composed from user’s `id`, and group’s `id` - users/USER\_ID/groups/GROUP\_ID.

For example:

```
$ kcadm.sh update users/b544f379-5fc4-49e5-8a8d-5cfb71f46f53/groups/ce01117a-7426-4670-a29a-5c118056fe20 -r demorealm -s realm=demorealm -s userId=b544f379-5fc4-49e5-8a8d-5cfb71f46f53 -s groupId=ce01117a-7426-4670-a29a-5c118056fe20 -n
```

Removing user from a group

To remove user from a group use `delete` operation against the same resource uri as used for adding user to a group - users/USER\_ID/groups/GROUP\_ID.

For example:

```
$ kcadm.sh delete users/b544f379-5fc4-49e5-8a8d-5cfb71f46f53/groups/ce01117a-7426-4670-a29a-5c118056fe20 -r demorealm
```

Listing assigned, available, and effective realm roles for a group

There is a dedicated 'get-roles' command to simplify listing of roles. It is an extension of `get` command thus it behaves like `get` command with additional semantics for listing roles.

To list **assigned** realm roles for the group you can specify the target group by `name` (via `--gname` option), `path` (via `--gpath` option), or `id` (via `--gid` option).

For example:

```
$ kcadm.sh get-roles -r demorealm --gname Group
```

To list **effective** realm roles, use additional `--effective` option.

For example:

```
$ kcadm.sh get-roles -r demorealm --gname Group --effective
```

To list realm roles that can still be added to the group, use `--available` option instead.

For example:

```
$ kcadm.sh get-roles -r demorealm --gname Group --available
```

Listing assigned, available, and effective client roles for a group

A dedicated 'get-roles' command can be used to list for both realm roles and client roles.

To list **assigned** client roles for the user you can specify the target group by either `name` (via --gname option) or `id` (via `--gid` option), and client by either `clientId` (via `--cclientid` option) or `id` (via `--id` option).

For example:

```
$ kcadm.sh get-roles -r demorealm --gname Group --cclientid realm-management
```

To list **effective** realm roles, use additional `--effective` option.

For example:

```
$ kcadm.sh get-roles -r demorealm --gname Group --cclientid realm-management --effective
```

To list realm roles that can still be added to the group, use `--available` option instead.

For example:

```
$ kcadm.sh get-roles -r demorealm --gname Group --cclientid realm-management --available
```

#### Identity Providers operations <a href="#identity_providers_operations" id="identity_providers_operations"></a>

Listing available identity providers

Use `serverinfo` endpoint to list available identity providers. For example:

```
$ kcadm.sh get serverinfo -r demorealm --fields 'identityProviders(*)'
```

Note that `serverinfo` endpoint is handled similarly to `realms` endpoint in that it is not resolved into resource URI as relative to target realm.

Listing configured identity providers

Use `identity-provider/instances` endpoint. For example:

```
$ kcadm.sh get identity-provider/instances -r demorealm --fields alias,providerId,enabled
```

Getting a specific configured identity provider

To get a specific identity provider use an `alias` attribute of identity provider to construct an endpoint uri - `identity-provider/instances/ALIAS`.

For example:

```
$ kcadm.sh get identity-provider/instances/facebook -r demorealm
```

Removing a specific configured identity provider

Use `delete` operation with the same endpoint uri as for getting a specific configured identity provider. For example:

```
$ kcadm.sh delete identity-provider/instances/facebook -r demorealm
```

Configuring a Keycloak OpenID Connect identity provider

For Keycloak OpenID Connect use `keycloak-oidc` as `providerId` when creating a new identity provider instance.

Provide config attributes `authorizationUrl`, `tokenUrl`, `clientId`, and `clientSecret`.

For example:

```
$ kcadm.sh create identity-provider/instances -r demorealm -s alias=keycloak-oidc -s providerId=keycloak-oidc -s enabled=true -s 'config.useJwksUrl="true"' -s config.authorizationUrl=http://localhost:8180/auth/realms/demorealm/protocol/openid-connect/auth -s config.tokenUrl=http://localhost:8180/auth/realms/demorealm/protocol/openid-connect/token -s config.clientId=demo-oidc-provider -s config.clientSecret=secret
```

Configuring an OpenID Connect identity provider

You configure the generic OpenID Connect provider the same way as Keycloak OpenID Connect provider, except that you set `providerId` attribute value to `oidc`.

Configuring a SAML 2 identity provider

Use `saml` as `providerId` when creating a new identity provider instance. Provide `config` attributes - `singleSignOnServiceUrl`, `nameIDPolicyFormat`, and `signatureAlgorithm`.

For example:

```
$ kcadm.sh create identity-provider/instances -r demorealm -s alias=saml -s providerId=saml -s enabled=true -s 'config.useJwksUrl="true"' -s config.singleSignOnServiceUrl=http://localhost:8180/auth/realms/saml-broker-realm/protocol/saml -s config.nameIDPolicyFormat=urn:oasis:names:tc:SAML:2.0:nameid-format:persistent -s config.signatureAlgorithm=RSA_SHA256
```

Configuring a Facebook identity provider

Use `facebook` as `providerId` when creating a new identity provider instance. Provide `config` attributes - `clientId` and `clientSecret` as obtained from Facebook Developers application configuration page for your application.

```
$ kcadm.sh create identity-provider/instances -r demorealm -s alias=facebook -s providerId=facebook -s enabled=true  -s 'config.useJwksUrl="true"' -s config.clientId=FACEBOOK_CLIENT_ID -s config.clientSecret=FACEBOOK_CLIENT_SECRET
```

Configuring a Google identity provider

Use `google` as `providerId` when creating a new identity provider instance. Provide `config` attributes - `clientId` and `clientSecret` as obtained from Google Developers application configuration page for your application.

```
$ kcadm.sh create identity-provider/instances -r demorealm -s alias=google -s providerId=google -s enabled=true  -s 'config.useJwksUrl="true"' -s config.clientId=GOOGLE_CLIENT_ID -s config.clientSecret=GOOGLE_CLIENT_SECRET
```

Configuring a Twitter identity provider

Use `twitter` as `providerId` when creating a new identity provider instance. Provide `config` attributes - `clientId` and `clientSecret` as obtained from Twitter Application Management application configuration page for your application.

```
$ kcadm.sh create identity-provider/instances -r demorealm -s alias=google -s providerId=google -s enabled=true  -s 'config.useJwksUrl="true"' -s config.clientId=TWITTER_API_KEY -s config.clientSecret=TWITTER_API_SECRET
```

Configuring a GitHub identity provider

Use `github` as `providerId` when creating a new identity provider instance. Provide `config` attributes - `clientId` and `clientSecret` as obtained from GitHub Developer Application Settings page for your application.

```
$ kcadm.sh create identity-provider/instances -r demorealm -s alias=github -s providerId=github -s enabled=true  -s 'config.useJwksUrl="true"' -s config.clientId=GITHUB_CLIENT_ID -s config.clientSecret=GITHUB_CLIENT_SECRET
```

Configuring a LinkedIn identity provider

Use `linkedin` as `providerId` when creating a new identity provider instance. Provide `config` attributes - `clientId` and `clientSecret` as obtained from LinkedIn Developer Console application page for your application.

```
$ kcadm.sh create identity-provider/instances -r demorealm -s alias=linkedin -s providerId=linkedin -s enabled=true  -s 'config.useJwksUrl="true"' -s config.clientId=LINKEDIN_CLIENT_ID -s config.clientSecret=LINKEDIN_CLIENT_SECRET
```

Configuring a Microsoft Live identity provider

Use `microsoft` as `providerId` when creating a new identity provider instance. Provide `config` attributes - `clientId` and `clientSecret` as obtained from Microsoft Application Registration Portal page for your application.

```
$ kcadm.sh create identity-provider/instances -r demorealm -s alias=microsoft -s providerId=microsoft -s enabled=true  -s 'config.useJwksUrl="true"' -s config.clientId=MICROSOFT_APP_ID -s config.clientSecret=MICROSOFT_PASSWORD
```

Configuring a StackOverflow identity provider

Use `stackoverflow` as `providerId` when creating a new identity provider instance. Provide `config` attributes - `clientId`, `clientSecret` and `key` as obtained from Stack Apps OAuth page for your application.

```
$ kcadm.sh create identity-provider/instances -r demorealm -s alias=stackoverflow -s providerId=stackoverflow -s enabled=true  -s 'config.useJwksUrl="true"' -s config.clientId=STACKAPPS_CLIENT_ID -s config.clientSecret=STACKAPPS_CLIENT_SECRET -s config.key=STACKAPPS_KEY
```

#### Storage Providers operations <a href="#storage_providers_operations" id="storage_providers_operations"></a>

Configuring a Kerberos storage provider

Use `create` against `user-federation/instances` endpoint. Specify `kerberos` as a value of `providerName` attribute.

For example:

```
$ kcadm.sh create user-federation/instances -r demorealm -s providerName=kerberos -s priority=0 -s config.debug=false -s config.allowPasswordAuthentication=true -s 'config.editMode="UNSYNCED"' -s config.updateProfileFirstLogin=true -s config.allowKerberosAuthentication=true -s 'config.kerberosRealm="KEYCLOAK.ORG"' -s 'config.keyTab="http.keytab"' -s 'config.serverPrincipal="HTTP/"'
```

Configuring an LDAP user storage provider

Use `create` against `components` endpoint. Specify `ldap` as a value of `providerId` attribute, and `org.keycloak.storage.UserStorageProvider` as value of `providerType` attribute. Provide realm `id` as value of `parentId` attribute.

For example, to create a Kerberos integrated LDAP provider:

```
$ kcadm.sh create components -r demorealm -s name=kerberos-ldap-provider -s providerId=ldap -s providerType=org.keycloak.storage.UserStorageProvider -s parentId=3d9c572b-8f33-483f-98a6-8bb421667867  -s 'config.priority=["1"]' -s 'config.fullSyncPeriod=["-1"]' -s 'config.changedSyncPeriod=["-1"]' -s 'config.cachePolicy=["DEFAULT"]' -s config.evictionDay=[] -s config.evictionHour=[] -s config.evictionMinute=[] -s config.maxLifespan=[] -s 'config.batchSizeForSync=["1000"]' -s 'config.editMode=["WRITABLE"]' -s 'config.syncRegistrations=["false"]' -s 'config.vendor=["other"]' -s 'config.usernameLDAPAttribute=["uid"]' -s 'config.rdnLDAPAttribute=["uid"]' -s 'config.uuidLDAPAttribute=["entryUUID"]' -s 'config.userObjectClasses=["inetOrgPerson, organizationalPerson"]' -s 'config.connectionUrl=["ldap://localhost:10389"]'  -s 'config.usersDn=["ou=People,dc=keycloak,dc=org"]' -s 'config.authType=["simple"]' -s 'config.bindDn=["uid=admin,ou=system"]' -s 'config.bindCredential=["secret"]' -s 'config.searchScope=["1"]' -s 'config.useTruststoreSpi=["ldapsOnly"]' -s 'config.connectionPooling=["true"]' -s 'config.pagination=["true"]' -s 'config.allowKerberosAuthentication=["true"]' -s 'config.serverPrincipal=["HTTP/"]' -s 'config.keyTab=["http.keytab"]' -s 'config.kerberosRealm=["KEYCLOAK.ORG"]' -s 'config.debug=["true"]' -s 'config.useKerberosForPasswordAuthentication=["true"]'
```

Removing a user storage provider instance

Use storage provider instance’s `id` attribute to compose an endpoint uri - `components/ID`.

Perform `delete` operation against this endpoint. For example:

```
$ kcadm.sh delete components/3d9c572b-8f33-483f-98a6-8bb421667867 -r demorealm
```

Triggering synchronization of all users for specific user storage provider

Use storage provider’s `id` attribute to compose an endpoint uri - user-storage/ID\_OF\_USER\_STORAGE\_INSTANCE/sync Add `action=triggerFullSync` query parameter and use `create`.

For example:

```
$ kcadm.sh create user-storage/b7c63d02-b62a-4fc1-977c-947d6a09e1ea/sync?action=triggerFullSync
```

Triggering synchronization of changed users for specific user storage provider

Use storage provider’s `id` attribute to compose an endpoint uri - user-storage/ID\_OF\_USER\_STORAGE\_INSTANCE/sync Add `action=triggerChangedUsersSync` query parameter and use `create`.

For example:

```
$ kcadm.sh create user-storage/b7c63d02-b62a-4fc1-977c-947d6a09e1ea/sync?action=triggerChangedUsersSync
```

Test LDAP user storage connectivity

Perform `get` operation against `testLDAPConnection` endpoint. Provide query parameters `bindCredential`, `bindDn`, `connectionUrl`, and `useTruststoreSpi`, and set `action` query parameter to `testConnection`.

For example:

```
$ kcadm.sh get testLDAPConnection -q action=testConnection -q bindCredential=secret -q bindDn=uid=admin,ou=system -q connectionUrl=ldap://localhost:10389 -q useTruststoreSpi=ldapsOnly
```

Test LDAP user storage authentication

Perform `get` operation against `testLDAPConnection` endpoint. Provide query parameters `bindCredential`, `bindDn`, `connectionUrl`, and `useTruststoreSpi`, and set `action` query parameter to `testAuthentication`.

For example:

```
$ kcadm.sh get testLDAPConnection -q action=testAuthentication -q bindCredential=secret -q bindDn=uid=admin,ou=system -q connectionUrl=ldap://localhost:10389 -q useTruststoreSpi=ldapsOnly
```

#### Adding mappers <a href="#adding_mappers" id="adding_mappers"></a>

Adding a hardcoded role LDAP mapper

Use `create` against `components` endpoint. Set `providerType` attribute to `org.keycloak.storage.ldap.mappers.LDAPStorageMapper`. Set `parentId` attribute to `id` of LDAP provider instance. Set `providerId` attribute to `hardcoded-ldap-role-mapper`. Make sure to provide a value of `role` config parameter.

For example:

```
$ kcadm.sh create components -r demorealm -s name=hardcoded-ldap-role-mapper -s providerId=hardcoded-ldap-role-mapper -s providerType=org.keycloak.storage.ldap.mappers.LDAPStorageMapper -s parentId=b7c63d02-b62a-4fc1-977c-947d6a09e1ea -s 'config.role=["realm-management.create-client"]'
```

Adding a MS Active Directory mapper

Use `create` against `components` endpoint. Set `providerType` attribute to `org.keycloak.storage.ldap.mappers.LDAPStorageMapper`. Set `parentId` attribute to `id` of LDAP provider instance. Set `providerId` attribute to `msad-user-account-control-mapper`.

For example:

```
$ kcadm.sh create components -r demorealm -s name=msad-user-account-control-mapper -s providerId=msad-user-account-control-mapper -s providerType=org.keycloak.storage.ldap.mappers.LDAPStorageMapper -s parentId=b7c63d02-b62a-4fc1-977c-947d6a09e1ea
```

Adding a user attribute LDAP mapper

Use `create` against `components` endpoint. Set `providerType` attribute to `org.keycloak.storage.ldap.mappers.LDAPStorageMapper`. Set `parentId` attribute to `id` of LDAP provider instance. Set `providerId` attribute to `user-attribute-ldap-mapper`.

For example:

```
$ kcadm.sh create components -r demorealm -s name=user-attribute-ldap-mapper -s providerId=user-attribute-ldap-mapper -s providerType=org.keycloak.storage.ldap.mappers.LDAPStorageMapper -s parentId=b7c63d02-b62a-4fc1-977c-947d6a09e1ea -s 'config."user.model.attribute"=["email"]' -s 'config."ldap.attribute"=["mail"]' -s 'config."read.only"=["false"]' -s 'config."always.read.value.from.ldap"=["false"]' -s 'config."is.mandatory.in.ldap"=["false"]'
```

Adding a group LDAP mapper

Use `create` against `components` endpoint. Set `providerType` attribute to `org.keycloak.storage.ldap.mappers.LDAPStorageMapper`. Set `parentId` attribute to `id` of LDAP provider instance. Set `providerId` attribute to `group-ldap-mapper`.

For example:

```
$ kcadm.sh create components -r demorealm -s name=group-ldap-mapper -s providerId=group-ldap-mapper -s providerType=org.keycloak.storage.ldap.mappers.LDAPStorageMapper -s parentId=b7c63d02-b62a-4fc1-977c-947d6a09e1ea -s 'config."groups.dn"=[]' -s 'config."group.name.ldap.attribute"=["cn"]' -s 'config."group.object.classes"=["groupOfNames"]' -s 'config."preserve.group.inheritance"=["true"]' -s 'config."membership.ldap.attribute"=["member"]' -s 'config."membership.attribute.type"=["DN"]' -s 'config."groups.ldap.filter"=[]' -s 'config.mode=["LDAP_ONLY"]' -s 'config."user.roles.retrieve.strategy"=["LOAD_GROUPS_BY_MEMBER_ATTRIBUTE"]' -s 'config."mapped.group.attributes"=["admins-group"]' -s 'config."drop.non.existing.groups.during.sync"=["false"]' -s 'config.roles=["admins"]' -s 'config.groups=["admins-group"]' -s 'config.group=[]' -s 'config.preserve=["true"]' -s 'config.membership=["member"]'
```

Adding a full name LDAP mapper

Use `create` against `components` endpoint. Set `providerType` attribute to `org.keycloak.storage.ldap.mappers.LDAPStorageMapper`. Set `parentId` attribute to `id` of LDAP provider instance. Set `providerId` attribute to `full-name-ldap-mapper`.

For example:

```
$ kcadm.sh create components -r demorealm -s name=full-name-ldap-mapper -s providerId=full-name-ldap-mapper -s providerType=org.keycloak.storage.ldap.mappers.LDAPStorageMapper -s parentId=b7c63d02-b62a-4fc1-977c-947d6a09e1ea -s 'config."ldap.full.name.attribute"=["cn"]' -s 'config."read.only"=["false"]' -s 'config."write.only"=["true"]'
```

#### Authentication operations <a href="#authentication_operations" id="authentication_operations"></a>

Setting a password policy

Set realm’s `passwordPolicy` attribute to an enumeration expression including specific policy provider id, and an optional configuration:

For example, to set password policy to 20000 hash iterations, requiring at least one special character, at least one uppercase character, at least one digit character, not be equal to user’s `username`, and be at least 8 characters long you would use the following:

```
$ kcadm.sh update realms/demorealm -s 'passwordPolicy="hashIterations and specialChars and upperCase and digits and notUsername and length"'
```

If you want want to use values different from defaults, pass configuration in brackets.

For example, to set password policy to 25000 hash iterations, requiring at least two special characters, at least two uppercase characters, at least two lowercase characters, at least two digits, be at least nine characters long, not be equal to user’s username, and not repeat for at least four changes back:

```
$ kcadm.sh update realms/demorealm -s 'passwordPolicy="hashIterations(25000) and specialChars(2) and upperCase(2) and lowerCase(2) and digits(2) and length(9) and notUsername and passwordHistory(4)"'
```

Getting the current password policy

Get current realm configuration and filter out everything but `passwordPolicy` attribute.

For example, to display `passwordPolicy` for demorealm:

```
$ kcadm.sh get realms/demorealm --fields passwordPolicy
```

Listing authentication flows

Use `get` operation against `authentication/flows` endpoint. For example:

```
$ kcadm.sh get authentication/flows -r demorealm
```

\
