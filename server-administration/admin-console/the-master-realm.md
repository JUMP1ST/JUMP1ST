# The Master Realm

When you boot Keycloak for the first time a pre-defined realm is created for you. This initial realm is called the _master_ realm and is the king of all realms. Admins in this realm have permissions to view and manage any other realm created on the server instance. When you define your initial admin account, you are creating an account in the _master_ realm. Your initial login to the admin console will also be through the _master_ realm.

It is recommended that you do not use the _master_ realm to manage the users and applications in your organization. Keep the _master_ realm as a place for _super_ admins to create and manage the realms in your system. This keeps things clean and organized.

It is possible to disable the _master_ realm and define admin accounts at each individual new realm you create. Each realm has its own dedicated Admin Console that you can log into with local accounts. This guide talks more about this in the [Dedicated Realm Admin Consoles](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/admin-console-permissions/per-realm.html#\_per\_realm\_admin\_permissions) chapter.
