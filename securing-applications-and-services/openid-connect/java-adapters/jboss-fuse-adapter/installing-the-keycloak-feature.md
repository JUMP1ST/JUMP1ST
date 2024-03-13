# Installing the Keycloak Feature

You must first install the `keycloak` feature in the JBoss Fuse environment. The keycloak feature includes the Fuse adapter and all third-party dependencies. You can install it either from the Maven repository or from an archive.

**Installing from the Maven Repository**

As a prequisite, you must be online and have access to the Maven repository.

For community it’s sufficient to be online as all the artifacts and 3rd party dependencies should be available in the maven central repository.

To install the keycloak feature using the Maven repository, complete the following steps:

1.  Start JBoss Fuse 6.3.0 Rollup 1; then in the Karaf terminal type:

    ```
    features:addurl mvn:org.keycloak/keycloak-osgi-features/SNAPSHOT/xml/features
    features:install keycloak
    ```
2.  You might also need to install the Jetty 9 feature:

    ```
    features:install keycloak-jetty9-adapter
    ```

    | Note | If you are using JBoss Fuse 6.2 or later, use `keycloak-jetty8-adapter`. However, upgrading to JBoss Fuse 6.3.0 Rollup 1 is recommended. |
    | ---- | ---------------------------------------------------------------------------------------------------------------------------------------- |
3. Ensure that the features were installed:

```
features:list | grep keycloak
```

**Installing from the ZIP bundle**

This is useful if you are offline or do not want to use Maven to obtain the JAR files and other artifacts.

To install the Fuse adapter from the ZIP archive, complete the following steps:

1. Download the Keycloak Fuse adapter ZIP archive.
2.  Unzip it into the root directory of JBoss Fuse. The dependencies are then installed under the `system` directory. You can overwrite all existing jar files.

    Use this for JBoss Fuse 6.3.0 Rollup 1:

    ```
    cd /path-to-fuse/jboss-fuse-6.3.0.redhat-254

    unzip -q /path-to-adapter-zip/keycloak-fuse-adapter-SNAPSHOT.zip
    ```
3.  Start Fuse and run these commands in the fuse/karaf terminal:

    ```
    features:addurl mvn:org.keycloak/keycloak-osgi-features/SNAPSHOT/xml/features
    features:install keycloak
    ```
4. Install the corresponding Jetty adapter. Since the artifacts are available directly in the JBoss Fuse `system` directory, you do not need to use the Maven repository.

\
