# Client Templates

If you have a lot of applications you need to secure and register within your organization it can become quite tedious to configure the [protocol mappers](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/clients/protocol-mappers.html#%3Cem%3Eprotocol-mappers) and [scope](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/roles/client-scope.html#\_client\_scope) for each of these clients. Keycloak allows you to define shared client configuration in an entity called a \_client template.

To create a client template, go to the `Client Templates` left menu item. This initial screen shows you a list of currently defined templates.

To create a template click the `Create` button. This brings you to a simple screen in which you name the template and hit save. A _client template_ will have similar tabs to regular clients. You’ll be able to define [protocol mappers](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/clients/protocol-mappers.html#\_protocol-mappers) and [scope](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/roles/client-scope.html#\_client\_scope) which can be inherited by other clients.

Having a client inherit from a template is as simple as choosing the template from the `Client Template` drop down list on either the `Add Client` or client `Settings` tab. You will see the `Mappers` and `Scope` tabs get additional switches which allow you to turn on or off inheriting from the parent template.

Future versions of client templating may get more inheritable configuration options, but for now, that’s all there is to talk about.

\
