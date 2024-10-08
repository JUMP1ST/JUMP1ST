# Import Implementation Strategy

When implementing a user storage provider, there’s another strategy you can take. Instead of using user federated storage, you can create a user locally in the Keycloak built-in user database and copy attributes from your external store into this local copy. There are many advantages to this approach.

* Keycloak basically becomes a persistence user cache for your external store. Once the user is imported you’ll no longer hit the external store thus taking load off of it.
* If you are moving to Keycloak as your official user store and deprecating the old external store, you can slowly migrate applications to use Keycloak. When all applications have been migrated, unlink the imported user, and retire the old legacy external store.

There are some obvious disadvantages though to using an import strategy:

* Looking up a user for the first time will require multiple updates to Keycloak database. This can be a big performance loss under load and put a lot of strain on the Keycloak database. The user federated storage approach will only store extra data as needed and may never be used depending on the capabilities of your external store.
* With the import approach, you have to keep local keycloak storage and external storage in sync. The User Storage SPI has capability interfaces that you can implement to support synchronization, but this can quickly become painful and messy.

To implement the import strategy you simply check to see first if the user has been imported locally. If so return the local user, if not create the user locally and import data from the external store. You can also proxy the local user so that most changes are automatically synchronized.

This will be a bit contrived, but we can extend our `PropertyFileUserStorageProvider` to take this approach. We begin first by modifying the `createAdapter()` method.

PropertyFileUserStorageProvider

```java
    protected UserModel createAdapter(RealmModel realm, String username) {
        UserModel local = session.userLocalStorage().getUserByUsername(username, realm);
        if (local == null) {
            local = session.userLocalStorage().addUser(realm, username);
            local.setFederationLink(model.getId());
        }
        return new UserModelDelegate(local) {
            @Override
            public void setUsername(String username) {
                String pw = (String)properties.remove(username);
                if (pw != null) {
                    properties.put(username, pw);
                    save();
                }
                super.setUsername(username);
            }
        };
    }
```

In this method we call the `KeycloakSession.userLocalStorage()` method to obtain a reference to local Keycloak user storage. We see if the user is stored locally, if not, we add it locally. Also note that we call `UserModel.setFederationLink()` and pass in the ID of the `ComponentModel` of our provider. This sets a link between the provider and the imported user.

| Note | When a user storage provider is removed, any user imported by it will also be removed. This is one of the purposes of calling `UserModel.setFederationLink()`. |
| ---- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |

Another thing to note is that if a local user is linked, your storage provider will still be delegated to for methods that it implements from the `CredentialInputValidator` and `CredentialInputUpdater` interfaces. Returning `false` from a validation or update will just result in Keycloak seeing if it can validate or update using local storage.

Also notice that we are proxying the local user using the ``org.keycloak.models.utils.UserModelDelegate' class. This class is an implementation of `UserModel``. Every method just delegates to the `UserModel` it was instantiated with. We override the `setUsername()` method of this delegate class to synchronize automatically with the property file. For your providers, you can use this to _intercept_ other methods on the local `UserModel` to perform synchronization with your external store. For example, get methods could make sure that the local store is in sync. Set methods keep the external store in sync with the local one.

| Note | If your provider is implementing the `UserRegistrationProvider` interface, your `removeUser()` method does not need to remove the user from local storage. The runtime will automatically perform this operation. Also note that `removeUser()` will be invoked before it is removed from local storage. |
| ---- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

**ImportedUserValidation Interface**

If you remember earlier in this chapter, we discussed how querying for a user worked. Local storage is queried first, if the user is found there, then the query ends. This is a problem for our above implementation as we want to proxy the local `UserModel` so that we can keep usernames in sync. The User Storage SPI has a callback for whenever a linked local user is loaded from the local database.

```java
package org.keycloak.storage.user;
public interface ImportedUserValidation {
    /**
     * If this method returns null, then the user in local storage will be removed
     *
     * @param realm
     * @param user
     * @return null if user no longer valid
     */
    UserModel validate(RealmModel realm, UserModel user);
}
```

Whenever a linked local user is loaded, if the user storage provider class implements this interface, then the `validate()` method is called. Here you can proxy the local user passed in as a parameter and return it. That new `UserModel` will be used. You can also optionally do a check to see if the user still exists in the external store. If `validate()` returns `null`, then the local user will be removed from the database.

**ImportSynchronization Interface**

With the import strategy you can see that it is possible for the local user copy to get out of sync with external storage. For example, maybe a user has been removed from the external store. The User Storage SPI has an additional interface you can implement to deal with this, `org.keycloak.storage.user.ImportSynchronization`:

```java
package org.keycloak.storage.user;

public interface ImportSynchronization {
    SynchronizationResult sync(KeycloakSessionFactory sessionFactory, String realmId, UserStorageProviderModel model);
    SynchronizationResult syncSince(Date lastSync, KeycloakSessionFactory sessionFactory, String realmId, UserStorageProviderModel model);
}
```

This interface is implemented by the provider factory. Once this interface is implemented by the provider factory, the administration console management page for the provider shows additional options. You can manually force a synchronization by clicking a button. This invokes the `ImportSynchronization.sync()` method. Also, additional configuration options are displayed that allow you to automatically schedule a synchronization. Automatic synchronizations invoke the `syncSince()` method.
