# Groups vs. Roles

In the IT world the concepts of Group and Role are often blurred and interchangeable. In Keycloak, Groups are just a collection of users that you can apply roles and attributes to in one place. Roles define a type of user and applications assign permission and access control to roles

Aren’t [Composite Roles](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/roles/composite.html#\_composite-roles) also similar to Groups? Logically they provide the same exact functionality, but the difference is conceptual. Composite roles should be used to apply the permission model to your set of services and applications. Groups should focus on collections of users and their roles in your organization. Use groups to manage users. Use composite roles to manage applications and services.
