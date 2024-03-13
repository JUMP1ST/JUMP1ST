# Profiles

Keycloak has a single profile, community, that enables all features by default, including features that are considered less mature. It is however possible to disable individual features.

The features that can be enabled and disabled are:

* Authorization - authorization services
* Docker - authentication protocol for Docker Registry
* Impersonation - ability for admins to impersonate users
* Script - write custom authenticators using JavaScript

To disable a specific feature start the server with:

```
bin/standalone.sh|bat -Dkeycloak.profile.feature.<feature name>=disabled
```

For example to disable Impersonation use `-Dkeycloak.profile.feature.impersonation=disabled`.

You can set this permanently in the `profile.properties` file by adding:

```
feature.impersonation=disabled
```
