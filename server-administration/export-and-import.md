# Export and Import

Keycloak has the ability to export and import the entire database. This can be especially useful if you want to migrate your whole Keycloak database from one environment to another or migrate to a different database (for example from MySQL to Oracle). Export and import is triggered at server boot time and its parameters are passed in via Java system properties. It is important to note that because import and export happens at server startup, no other actions should be taken on the server or the database while this happens.

You can export/import your database either to:

* Directory on local filesystem
* Single JSON file on your filesystem

When importing using the directory strategy, note that the files need to follow the naming convention specified below. If you are importing files which were previously exported, the files already follow this convention.

* {REALM\_NAME}-realm.json, such as "acme-roadrunner-affairs-realm.json" for the realm named "acme-roadrunner-affairs"
* {REALM\_NAME}-users-{INDEX}.json, such as "acme-roadrunner-affairs-users-0.json" for the first users file of the realm named "acme-roadrunner-affairs"

If you export to a directory, you can also specify the number of users that will be stored in each JSON file.

| Note | If you have bigger amount of users in your database (500 or more), it’s highly recommended to export into directory rather than to single file. Exporting into single file may lead to the very big file. Also the directory provider is using separate transaction for each "page" (file with users), which leads to much better performance. Default count of users per file (and transaction) is 50, which showed us best performance, but you have possibility to override (See below). Exporting to single file is using one transaction per whole export and one per whole import, which results in bad performance with large amount of users. |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

To export into unencrypted directory you can use:

```
bin/standalone.sh -Dkeycloak.migration.action=export
-Dkeycloak.migration.provider=dir -Dkeycloak.migration.dir=<DIR TO EXPORT TO>
```

And similarly for import just use `-Dkeycloak.migration.action=import` instead of `export` . To export into single JSON file you can use:

```
bin/standalone.sh -Dkeycloak.migration.action=export
-Dkeycloak.migration.provider=singleFile -Dkeycloak.migration.file=<FILE TO EXPORT TO>
```

Here’s an example of importing:

```
bin/standalone.sh -Dkeycloak.migration.action=import
-Dkeycloak.migration.provider=singleFile -Dkeycloak.migration.file=<FILE TO IMPORT>
-Dkeycloak.migration.strategy=OVERWRITE_EXISTING
```

Other available options are:

\-Dkeycloak.migration.realmName

This property is used if you want to export just one specified realm instead of all. If not specified, then all realms will be exported.

\-Dkeycloak.migration.usersExportStrategy

This property is used to specify where users are exported. Possible values are:

* DIFFERENT\_FILES - Users will be exported into different files according to the maximum number of users per file. This is default value.
* SKIP - Exporting of users will be skipped completely.
* REALM\_FILE - All users will be exported to same file with the realm settings. (The result will be a file like "foo-realm.json" with both realm data and users.)
* SAME\_FILE - All users will be exported to same file but different from the realm file. (The result will be a file like "foo-realm.json" with realm data and "foo-users.json" with users.)

\-Dkeycloak.migration.usersPerFile

This property is used to specify the number of users per file (and also per DB transaction). It’s 50 by default. It’s used only if usersExportStrategy is DIFFERENT\_FILES

\-Dkeycloak.migration.strategy

This property is used during import. It can be used to specify how to proceed if a realm with same name already exists in the database where you are going to import data. Possible values are:

* IGNORE\_EXISTING - Ignore importing if a realm of this name already exists.
* OVERWRITE\_EXISTING - Remove existing realm and import it again with new data from the JSON file. If you want to fully migrate one environment to another and ensure that the new environment will contain the same data as the old one, you can specify this.

When importing realm files that weren’t exported before, the option `keycloak.import` can be used. If more than one realm file needs to be imported, a comma separated list of file names can be specified. This is more appropriate than the cases before, as this will happen only after the master realm has been initialized. Examples:

* \-Dkeycloak.import=/tmp/realm1.json
* \-Dkeycloak.import=/tmp/realm1.json,/tmp/realm2.json

#### Admin console export/import <a href="#admin_console_export_import" id="admin_console_export_import"></a>

Import of most resources can be performed from the admin console as well as export of most resources. Export of users is not supported.

Note: Attributes containing secrets or private information will be masked in export file.

The files created during a "startup" export can also be used to import from the admin UI. This way, you can export from one realm and import to another realm. Or, you can export from one server and import to another. Note: The admin console export/import allows just one realm per file.

| Warning | The admin console import allows you to "overwrite" resources if you choose. Use this feature with caution, especially on a production system. |
| ------- | --------------------------------------------------------------------------------------------------------------------------------------------- |

| Warning | The admin console export allows you to export clients, groups, and roles. If there is a great number of any of these assets in your realm, the operation may take some time to complete. During that time server may not be responsive to user requests. Use this feature with caution, especially on a production system. |
| ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
