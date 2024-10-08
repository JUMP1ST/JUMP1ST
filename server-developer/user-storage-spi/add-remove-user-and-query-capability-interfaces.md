# Add/Remove User and Query Capability interfaces

One thing we have not done with our example is allow it to add and remove users or change passwords. Users defined in our example are also not queryable or viewable in the administration console. To add these enhancements, our example provider must implement the `UserQueryProvider` and `UserRegistrationProvider` interfaces.

**Implementing UserRegistrationProvider**

To implement adding and removing users from this particular store, we first have to be able to save our properties file to disk.

PropertyFileUserStorageProvider

```java
    public void save() {
        String path = model.getConfig().getFirst("path");
        path = EnvUtil.replace(path);
        try {
            FileOutputStream fos = new FileOutputStream(path);
            properties.store(fos, "");
            fos.close();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
```

Then, the implementation of the `addUser()` and `removeUser()` methods becomes simple.

PropertyFileUserStorageProvider

```java
    public static final String UNSET_PASSWORD="#$!-UNSET-PASSWORD";

    @Override
    public UserModel addUser(RealmModel realm, String username) {
        synchronized (properties) {
            properties.setProperty(username, UNSET_PASSWORD);
            save();
        }
        return createAdapter(realm, username);
    }

    @Override
    public boolean removeUser(RealmModel realm, UserModel user) {
        synchronized (properties) {
            if (properties.remove(user.getUsername()) == null) return false;
            save();
            return true;
        }
    }
```

Notice that when adding a user we set the password value of the property map to be `UNSET_PASSWORD`. We do this as we can’t have null values for a property in the property value. We also have to modify the `CredentialInputValidator` methods to reflect this.

`addUser()` will be called if the provider implements the `UserRegistrationProvider` interface. If your provider has a configuration switch to turn of adding a user, returning `null` from this method will skip the provider and call the next one.

PropertyFileUserStorageProvider

```java
    @Override
    public boolean isValid(RealmModel realm, UserModel user, CredentialInput input) {
        if (!supportsCredentialType(input.getType()) || !(input instanceof UserCredentialModel)) return false;

        UserCredentialModel cred = (UserCredentialModel)input;
        String password = properties.getProperty(user.getUsername());
        if (password == null || UNSET_PASSWORD.equals(password)) return false;
        return password.equals(cred.getValue());
    }
```

Since we can now save our property file, it also makes sense to allow password updates.

PropertyFileUserStorageProvider

```java
    @Override
    public boolean updateCredential(RealmModel realm, UserModel user, CredentialInput input) {
        if (!(input instanceof UserCredentialModel)) return false;
        if (!input.getType().equals(CredentialModel.PASSWORD)) return false;
        UserCredentialModel cred = (UserCredentialModel)input;
        synchronized (properties) {
            properties.setProperty(user.getUsername(), cred.getValue());
            save();
        }
        return true;
    }
```

We can now also implement disabling a password too.

PropertyFileUserStorageProvider

```java
    @Override
    public void disableCredentialType(RealmModel realm, UserModel user, String credentialType) {
        if (!credentialType.equals(CredentialModel.PASSWORD)) return;
        synchronized (properties) {
            properties.setProperty(user.getUsername(), UNSET_PASSWORD);
            save();
        }

    }

    private static final Set<String> disableableTypes = new HashSet<>();

    static {
        disableableTypes.add(CredentialModel.PASSWORD);
    }

    @Override
    public Set<String> getDisableableCredentialTypes(RealmModel realm, UserModel user) {

        return disableableTypes;
    }
```

With these methods implemented, you’ll now be able to change and disable the password for the user in the administration console.

**Implementing UserQueryProvider**

Without implementing `UserQueryProvider` the administration console would not be able to view and manage users that were loaded by our example provider. Let’s look at implementing this interface.

PropertyFileUserStorageProvider

```java
    @Override
    public int getUsersCount(RealmModel realm) {
        return properties.size();
    }

    @Override
    public List<UserModel> getUsers(RealmModel realm) {
        return getUsers(realm, 0, Integer.MAX_VALUE);
    }

    @Override
    public List<UserModel> getUsers(RealmModel realm, int firstResult, int maxResults) {
        List<UserModel> users = new LinkedList<>();
        int i = 0;
        for (Object obj : properties.keySet()) {
            if (i++ < firstResult) continue;
            String username = (String)obj;
            UserModel user = getUserByUsername(username, realm);
            users.add(user);
            if (users.size() >= maxResults) break;
        }
        return users;
    }
```

The `getUser()` method iterates over the key set of the property file, delegating to `getUserByUsername()` to load a user. Notice that we are indexing this call based on the `firstResult` and `maxResults` parameter. If your external store does not support pagination, you will have to do similar logic.

PropertyFileUserStorageProvider

```java
    @Override
    public List<UserModel> searchForUser(String search, RealmModel realm) {
        return searchForUser(search, realm, 0, Integer.MAX_VALUE);
    }

    @Override
    public List<UserModel> searchForUser(String search, RealmModel realm, int firstResult, int maxResults) {
        List<UserModel> users = new LinkedList<>();
        int i = 0;
        for (Object obj : properties.keySet()) {
            String username = (String)obj;
            if (!username.contains(search)) continue;
            if (i++ < firstResult) continue;
            UserModel user = getUserByUsername(username, realm);
            users.add(user);
            if (users.size() >= maxResults) break;
        }
        return users;
    }
```

The first declaration of `searchForUser()` takes a string parameter. This is supposed to be a string that you use to search username and email attributes to find the user. This string can be a substring, which is why we use the `String.contains()` method when doing our search.

PropertyFileUserStorageProvider

```java
    @Override
    public List<UserModel> searchForUser(Map<String, String> params, RealmModel realm) {
        return searchForUser(params, realm, 0, Integer.MAX_VALUE);
    }

    @Override
    public List<UserModel> searchForUser(Map<String, String> params, RealmModel realm, int firstResult, int maxResults) {
        // only support searching by username
        String usernameSearchString = params.get("username");
        if (usernameSearchString == null) return Collections.EMPTY_LIST;
        return searchForUser(usernameSearchString, realm, firstResult, maxResults);
    }
```

The `searchForUser()` method that takes a `Map` parameter can search for a user based on first, last name, username, and email. We only store usernames, so we only search based on usernames. We delegate to `searchForUser()` for this.

PropertyFileUserStorageProvider

```java
    @Override
    public List<UserModel> getGroupMembers(RealmModel realm, GroupModel group, int firstResult, int maxResults) {
        return Collections.EMPTY_LIST;
    }

    @Override
    public List<UserModel> getGroupMembers(RealmModel realm, GroupModel group) {
        return Collections.EMPTY_LIST;
    }

    @Override
    public List<UserModel> searchForUserByUserAttribute(String attrName, String attrValue, RealmModel realm) {
        return Collections.EMPTY_LIST;
    }
```

We do not store groups or attributes, so the other methods return an empty list.
