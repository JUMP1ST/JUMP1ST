# Model Interfaces

Most of the methods defined in the _capability_ _interfaces_ either return or are passed in representations of a user. These representations are defined by the `org.keycloak.models.UserModel` interface. App developers are required to implement this interface. It provides a mapping between the external user store and the user metamodel that Keycloak uses.

```java
package org.keycloak.models;

public interface UserModel extends RoleMapperModel {
    String getId();

    String getUsername();
    void setUsername(String username);

    String getFirstName();
    void setFirstName(String firstName);

    String getLastName();
    void setLastName(String lastName);

    String getEmail();
    void setEmail(String email);
...
}
```

`UserModel` implementations provide access to read and update metadata about the user including things like username, name, email, role and group mappings, as well as other arbitrary attributes.

There are other model classes within the `org.keycloak.models` package that represent other parts of the Keycloak metamodel: `RealmModel`, `RoleModel`, `GroupModel`, and `ClientModel`.

**Storage Ids**

One important method of `UserModel` is the `getId()` method. When implementing `UserModel` developers must be aware of the user id format. The format must be:

```
"f:" + component id + ":" + external id
```

The Keycloak runtime often has to look up users by their user id. The user id contains enough information so that the runtime does not have to query every single `UserStorageProvider` in the system to find the user.

The component id is the id returned from `ComponentModel.getId()`. The `ComponentModel` is passed in as a parameter when creating the provider class so you can get it from there. The external id is information your provider class needs to find the user in the external store. This is often a username or a uid. For example, it might look something like this:

```
f:332a234e31234:wburke
```

When the runtime does a lookup by id, the id is parsed to obtain the component id. The component id is used to locate the `UserStorageProvider` that was originally used to load the user. That provider is then passed the id. The provider again parses the id to obtain the external id it will use to locate the user in external user storage.

\
