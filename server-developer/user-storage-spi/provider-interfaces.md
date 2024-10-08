# Provider Interfaces

When building an implementation of the User Storage SPI you have to define a provider class and a provider factory. Provider class instances are created per transaction by provider factories. Provider classes do all the heavy lifting of user lookup and other user operations. They must implement the `org.keycloak.storage.UserStorageProvider` interface.

```java
package org.keycloak.storage;

public interface UserStorageProvider extends Provider {


    /**
     * Callback when a realm is removed.  Implement this if, for example, you want to do some
     * cleanup in your user storage when a realm is removed
     *
     * @param realm
     */
    default
    void preRemove(RealmModel realm) {

    }

    /**
     * Callback when a group is removed.  Allows you to do things like remove a user
     * group mapping in your external store if appropriate
     *
     * @param realm
     * @param group
     */
    default
    void preRemove(RealmModel realm, GroupModel group) {

    }

    /**
     * Callback when a role is removed.  Allows you to do things like remove a user
     * role mapping in your external store if appropriate

     * @param realm
     * @param role
     */
    default
    void preRemove(RealmModel realm, RoleModel role) {

    }

}
```

You may be thinking that the `UserStorageProvider` interface is pretty sparse? You’ll see later in this chapter that there are other mix-in interfaces your provider class may implement to support the meat of user integration.

`UserStorageProvider` instances are created once per transaction. When the transaction is complete, the `UserStorageProvider.close()` method is invoked and the instance is then garbage collected. Instances are created by provider factories. Provider factories implement the `org.keycloak.storage.UserStorageProviderFactory` interface.

```java
package org.keycloak.storage;

/**
 * @author <a href="mailto:bill@burkecentral.com">Bill Burke</a>
 * @version $Revision: 1 $
 */
public interface UserStorageProviderFactory<T extends UserStorageProvider> extends ComponentFactory<T, UserStorageProvider> {

    /**
     * This is the name of the provider and will be showed in the admin console as an option.
     *
     * @return
     */
    @Override
    String getId();

    /**
     * called per Keycloak transaction.
     *
     * @param session
     * @param model
     * @return
     */
    T create(KeycloakSession session, ComponentModel model);
...
}
```

Provider factory classses must specify the concrete provider class as a template parameter when implementing the `UserStorageProviderFactory`. This is a must as the runtime will introspect this class to scan for its capabilities (the other interfaces it implements). So for example, if your provider class is named `FileProvider`, then the factory class should look like this:

```java
public class FileProviderFactory implements UserStorageProviderFactory<FileProvider> {

    public String getId() { return "file-provider"; }

    public FileProvider create(KeycloakSession session, ComponentModel model) {
       ...
    }
```

The `getId()` method returns the name of the User Storage provider. This id will be displayed in the admin console’s `UserFederation` page when you want to enable the provider for a specific realm.

The `create()` method is responsible for allocating an instance of the provider class. It takes a `org.keycloak.models.KeycloakSession` parameter. This object can be used to lookup other information and metadata as well as provide access to various other components within the runtime. The `ComponentModel` parameter represents how the provider was enabled and configured within a specific realm. It contains the instance id of the enabled provider as well as any configuration you may have specified for it when you enabled through the admin console.

The `UserStorageProviderFactory` has other capabilities as well which we will go over later in this chapter.
