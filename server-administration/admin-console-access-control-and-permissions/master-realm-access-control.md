# Master Realm Access Control

The `master` realm in Keycloak is a special realm and treated differently than other realms. Users in the Keycloak `master` realm can be granted permission to manage zero or more realms that are deployed on the Keycloak server. When a realm is created, Keycloak automatically creates various roles that grant fine-grain permissions to access that new realm. Access to The Admin Console and Admin REST endpoints can be controlled by mapping these roles to users in the `master` realm. It’s possible to create multiple super users, as well as users that can only manage specific realms.

**Global Roles**

There are two realm-level roles in the `master` realm. These are:

* admin
* create-realm

Users with the `admin` role are super users and have full access to manage any realm on the server. Users with the `create-realm` role are allowed to create new realms. They will be granted full access to any new realm they create.

**Realm Specific Roles**

Admin users within the `master` realm can be granted management privileges to one or more other realms in the system. Each realm in Keycloak is represented by a client in the `master` realm. The name of the client is `<realm name>-realm`. These clients each have client-level roles defined which define varying level of access to manage an individual realm.

The roles available are:

* view-realm
* view-users
* view-clients
* view-events
* manage-realm
* manage-users
* create-client
* manage-clients
* manage-events
* view-identity-providers
* manage-identity-providers
* impersonation

Assign the roles you want to your users and they will only be able to use that specific part of the administration console.
