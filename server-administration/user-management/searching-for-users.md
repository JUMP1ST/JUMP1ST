# Searching For Users

If you need to manage a specific user, click on `Users` in the left menu bar.

Users

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

This menu option brings you to the user list page. In the search box you can type in a full name, last name, or email address you want to search for in the user database. The query will bring up all users that match your criteria. The `View all users` button will list every user in the system. This will search just local Keycloak database and not the federated database (ie. LDAP) because some backends like LDAP don’t have a way to page through users. So if you want the users from federated backend to be synced into Keycloak database you need to either:

* Adjust search criteria. That will sync just the backend users matching the criteria into Keycloak database.
* Go to `User Federation` tab and click `Sync all users` or `Sync changed users` in the page with your federation provider.

See [User Federation](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/user-federation.html#\_user-storage-federation) for more details.

\
