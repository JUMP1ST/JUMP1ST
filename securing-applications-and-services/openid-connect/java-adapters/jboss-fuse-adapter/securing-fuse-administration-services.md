# Securing Fuse Administration Services

**Using SSH Authentication to Fuse Terminal**

Keycloak mainly addresses use cases for authentication of web applications; however, if your other web services and applications are protected with Keycloak, protecting non-web administration services such as SSH with Keycloak credentials is a best pracrice. You can do this using the JAAS login module, which allows remote connection to Keycloak and verifies credentials based on [Resource Owner Password Credentials](https://wjw465150.gitbooks.io/keycloak-documentation/content/securing\_apps/topics/oidc/oidc-generic.html#\_resource\_owner\_password\_credentials\_flow).

To enable SSH authentication, complete the following steps:

1. In Keycloak create a client (for example, `ssh-jmx-admin-client`), which will be used for SSH authentication. This client needs to have `Direct Access Grants Enabled` selected to `On`.
2.  In the `$FUSE_HOME/etc/org.apache.karaf.shell.cfg` file, update or specify this property:

    ```
    sshRealm=keycloak
    ```
3.  Add the `$FUSE_HOME/etc/keycloak-direct-access.json` file with content similar to the following (based on your environment and Keycloak client settings):

    ```json
    {
        "realm": "demo",
        "resource": "ssh-jmx-admin-client",
        "ssl-required" : "external",
        "auth-server-url" : "http://localhost:8080/auth",
        "credentials": {
            "secret": "password"
        }
    }
    ```

    This file specifies the client application configuration, which is used by JAAS DirectAccessGrantsLoginModule from the `keycloak` JAAS realm for SSH authentication.
4.  Start Fuse and install the `keycloak` JAAS realm. The easiest way is to install the `keycloak-jaas` feature, which has the JAAS realm predefined. You can override the feature’s predefined realm by using your own `keycloak` JAAS realm with higher ranking. For details see the https:access.redhat.com/documentation/en-us/red\_hat\_jboss\_fuse/6.3/html-single/security\_guide/#ESBSecureContainer\[JBoss Fuse documentation].

    Use these commands in the Fuse terminal:

    ```
    features:addurl mvn:org.keycloak/keycloak-osgi-features/SNAPSHOT/xml/features
    features:install keycloak-jaas
    ```
5.  Log in using SSH as `admin` user by typing the following in the terminal:

    ```
    ssh -o PubkeyAuthentication=no -p 8101 admin@localhost
    ```
6. Log in with password `password`.

| Note | On some later operating systems, you might also need to use the SSH command’s -o option `-o HostKeyAlgorithms=+ssh-dss` because later SSH clients do not allow use of the `ssh-dss` algorithm, by default. However, by default, it is currently used in JBoss Fuse 6.3.0 Rollup 1. |
| ---- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

Note that the user needs to have realm role `admin` to perform all operations or another role to perform a subset of operations (for example, the **viewer** role that restricts the user to run only read-only Karaf commands). The available roles are configured in `$FUSE_HOME/etc/org.apache.karaf.shell.cfg` or `$FUSE_HOME/etc/system.properties`.

**Using JMX Authentication**

JMX authentication might be necessary if you want to use jconsole or another external tool to remotely connect to JMX through RMI. Otherwise it might be better to use hawt.io/jolokia, since the jolokia agent is installed in hawt.io by default. For more details see [Hawtio Admin Console](https://wjw465150.gitbooks.io/keycloak-documentation/content/securing\_apps/topics/oidc/java/fuse/hawtio.html#\_hawtio).

To use JMX authentication, complete the following steps:

1.  In the `$FUSE_HOME/etc/org.apache.karaf.management.cfg` file, change the jmxRealm property to:

    ```
    jmxRealm=keycloak
    ```
2. Install the `keycloak-jaas` feature and configure the `$FUSE_HOME/etc/keycloak-direct-access.json` file as described in the SSH section above.
3. In jconsole you can use a URL such as:

```
service:jmx:rmi://localhost:44444/jndi/rmi://localhost:1099/karaf-root
```

and credentials: admin/password (based on the user with admin privileges according to your environment).
