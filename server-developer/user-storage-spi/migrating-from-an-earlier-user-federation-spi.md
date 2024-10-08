# Migrating from an Earlier User Federation SPI

| Note | This chapter is only applicable if you have implemented a provider using the earlier (and now removed) User Federation SPI. |
| ---- | --------------------------------------------------------------------------------------------------------------------------- |

In Keycloak version 2.4.0 and earlier there was a User Federation SPI. Red Hat Single Sign-On version 7.0, although unsupported, also had this earlier SPI available as well. This earlier User Federation SPI has been removed from Keycloak version 2.5.0 and Red Hat Single Sign-On version 7.1. However, if you have written a provider with this earlier SPI, this chapter discusses some strategies you can use to port it.

**Import vs. Non-Import**

The earlier User Federation SPI required you to create a local copy of a user in the Keycloak’s database and import information from your external store to the local copy. However, this is no longer a requirement. You can still port your earlier provider as-is, but you should consider whether a non-import strategy might be a better approach.

Advantages of the import strategy:

* Keycloak basically becomes a persistence user cache for your external store. Once the user is imported you’ll no longer hit the external store, thus taking load off of it.
* If you are moving to Keycloak as your official user store and deprecating the earlier external store, you can slowly migrate applications to use Keycloak. When all applications have been migrated, unlink the imported user, and retire the earlier legacy external store.

There are some obvious disadvantages though to using an import strategy:

* Looking up a user for the first time will require multiple updates to Keycloak database. This can be a big performance loss under load and put a lot of strain on the Keycloak database. The user federated storage approach will only store extra data as needed and might never be used depending on the capabilities of your external store.
* With the import approach, you have to keep local keycloak storage and external storage in sync. The User Storage SPI has capability interfaces that you can implement to support synchronization, but this can quickly become painful and messy.

**UserFederationProvider vs. UserStorageProvider**

The first thing to notice is that `UserFederationProvider` was a complete interface. You implemented every method in this interface. However, `UserStorageProvider` has instead broken up this interface into multiple capability interfaces that you implement as needed.

`UserFederationProvider.getUserByUsername()` and `getUserByEmail()` have exact equivalents in the new SPI. The difference between the two is how you import. If you are going to continue with an import strategy, you no longer call `KeycloakSession.userStorage().addUser()` to create the user locally. Instead you call `KeycloakSession.userLocalStorage().addUser()`. The `userStorage()` method no longer exists.

The `UserFederationProvider.validateAndProxy()` method has been moved to an optional capability interface, `ImportedUserValidation`. You want to implement this interface if you are porting your earlier provider as-is. Also note that in the earlier SPI, this method was called every time the user was accessed, even if the local user is in the cache. In the later SPI, this method is only called when the local user is loaded from local storage. If the local user is cached, then the `ImportedUserValidation.validate()` method is not called at all.

The `UserFederationProvider.isValid()` method no longer exists in the later model.

The `UserFederationProvider` methods `synchronizeRegistrations()`, `registerUser()`, and `removeUser()` have been moved to the `UserRegistrationProvider` capability interface. This new interface is optional to implement so if your provider does not support creating and removing users, you don’t have to implement it. If your earlier provider had switch to toggle support for registering new users, this is supported in the new SPI, returning `null` from `UserRegistrationProvider.addUser()` if the provider doesn’t support adding users.

The earlier `UserFederationProvider` methods centered around credentials are now encapsulated in the `CredentialInputValidator` and `CredentialInputUpdater` interfaces, which are also optional to implement depending on if you support validating or updating credentials. Credential management used to exist in `UserModel` methods. These also have been moved to the `CredentialInputValidator` and `CredentialInputUpdater` interfaces. One thing to note that if you do not implement the `CredentialInputUpdater` interface, then any credentials provided by your provider can be overridden locally in Keycloak storage. So if you want your credentials to be read-only, implement the `CredentialInputUpdater.updateCredential()` method and return a `ReadOnlyException`.

The `UserFederationProvider` query methods such as `searchByAttributes()` and `getGroupMembers()` are now encapsulated in an optional interface `UserQueryProvider`. If you do not implement this interface, then users will not be viewable in the admin console. You’ll still be able to login though.

**UserFederationProviderFactory vs. UserStorageProviderFactory**

The synchronization methods in the earlier SPI are now encapsulated within an optional `ImportSynchronization` interface. If you have implemented synchronization logic, then have your new `UserStorageProviderFactory` implement the `ImportSynchronization` interface.

**Upgrading to a New Model**

The User Storage SPI instances are stored in a different set of relational tables. Keycloak automatically runs a migration script. If any earlier User Federation providers are deployed for a realm, they are converted to the later storage model as is, including the `id` of the data. This migration will only happen if a User Storage provider exists with the same provider ID (i.e., "ldap", "kerberos") as the earlier User Federation provider.

So, knowing this there are different approaches you can take.

1. You can remove the earlier provider in your earlier Keycloak deployment. This will remove all local linked copies of imported users. Then, when you upgrade Keycloak, just deploy and configure your new provider for your realm.
2. The second option is to write your new provider making sure it has the same provider ID: `UserStorageProviderFactory.getId()`. Make sure this provider is in the `deploy/` directory of the new Keycloak installation. Boot the server, and have the built-in migration script convert from the earlier data model to the later data model. In this case all your earlier linked imported users will work and be the same.

If you have decided to get rid of the import strategy and rewrite your User Storage provider, we suggest that you remove the earlier provider before upgrading Keycloak. This will remove linked local imported copies of any user you imported.
