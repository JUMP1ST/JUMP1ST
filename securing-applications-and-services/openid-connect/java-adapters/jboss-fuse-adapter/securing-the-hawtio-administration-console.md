# Securing the Hawtio Administration Console

To secure the Hawtio Administration Console with Keycloak, complete the following steps:

1.  Add these properties to the `$FUSE_HOME/etc/system.properties` file:

    ```
    hawtio.keycloakEnabled=true
    hawtio.realm=keycloak
    hawtio.keycloakClientConfig=${karaf.base}/etc/keycloak-hawtio-client.json
    hawtio.rolePrincipalClasses=org.keycloak.adapters.jaas.RolePrincipal,org.apache.karaf.jaas.boot.principal.RolePrincipal
    ```
2. Create a client in the Keycloak administration console in your realm. For example, in the Keycloak `demo` realm, create a client `hawtio-client`, specify `public` as the Access Type, and specify a redirect URI pointing to Hawtio: http://localhost:8181/hawtio/\*. You must also have a corresponding Web Origin configured (in this case, http://localhost:8181).
3.  Create the `keycloak-hawtio-client.json` file in the `$FUSE_HOME/etc` directory using content similar to that shown in the example below. Change the `realm`, `resource`, and `auth-server-url` properties according to your Keycloak environment. The `resource` property must point to the client created in the previous step. This file is used by the client (Hawtio Javascript application) side.

    ```json
    {
      "realm" : "demo",
      "resource" : "hawtio-client",
      "auth-server-url" : "http://localhost:8080/auth",
      "ssl-required" : "external",
      "public-client" : true
    }
    ```
4.  Create the `keycloak-hawtio.json` file in the `$FUSE_HOME/etc` dicrectory using content similar to that shown in the example below. Change the `realm` and `auth-server-url` properties according to your Keycloak environment. This file is used by the adapters on the server (JAAS Login module) side.

    ```json
    {
      "realm" : "demo",
      "resource" : "jaas",
      "bearer-only" : true,
      "auth-server-url" : "http://localhost:8080/auth",
      "ssl-required" : "external",
      "use-resource-role-mappings": false,
      "principal-attribute": "preferred_username"
    }
    ```
5.  Start JBoss Fuse 6.3.0 Rollup 1 and install the keycloak feature if you have not already done so. The commands in Karaf terminal are similar to this example:

    ```
    features:addurl mvn:org.keycloak/keycloak-osgi-features/SNAPSHOT/xml/features
    features:install keycloak
    ```
6.  Go to [http://localhost:8181/hawtio](http://localhost:8181/hawtio) and log in as a user from your Keycloak realm.

    Note that the user needs to have the proper realm role to successfully authenticate to Hawtio. The available roles are configured in the `$FUSE_HOME/etc/system.properties` file in `hawtio.roles`.

**Securing Hawtio on Wildfly 10**

To run Hawtio on the Wildfly 10 server, complete the following steps:

1. Set up Keycloak as described in the previous section, Securing the Hawtio Administration Console. It is assumed that:
   * you have a Keycloak realm `demo` and client `hawtio-client`
   * your Keycloak is running on `localhost:8080`
   * the Wildfly 10 server with deployed Hawtio will be running on `localhost:8181`. The directory with this server is referred in next steps as `$EAP_HOME`.
2. Copy the `hawtio-wildfly-1.4.0.redhat-630254.war` archive to the `$EAP_HOME/standalone/configuration` directory. For more details about deploying Hawtio see the [Fuse Hawtio documentation](https://access.redhat.com/documentation/en-us/red\_hat\_jboss\_fuse/6.3/html-single/deploying\_into\_a\_web\_server/eapcamelsubsystem#idm140313338064000).
3. Copy the `keycloak-hawtio.json` and `keycloak-hawtio-client.json` files with the above content to the `$EAP_HOME/standalone/configuration` directory.
4. Install the Keycloak adapter subsystem to your Wildfly 10 server as described in the [JBoss adapter documentation](https://wjw465150.gitbooks.io/keycloak-documentation/content/securing\_apps/topics/oidc/java/jboss-adapter.html#\_jboss\_adapter).
5.  In the `$EAP_HOME/standalone/configuration/standalone.xml` file configure the system properties as in this example:

    ```xml
    <extensions>
    ...
    </extensions>

    <system-properties>
        <property name="hawtio.authenticationEnabled" value="true" />
        <property name="hawtio.realm" value="hawtio" />
        <property name="hawtio.roles" value="admin,viewer" />
        <property name="hawtio.rolePrincipalClasses" value="org.keycloak.adapters.jaas.RolePrincipal" />
        <property name="hawtio.keycloakEnabled" value="true" />
        <property name="hawtio.keycloakClientConfig" value="${jboss.server.config.dir}/keycloak-hawtio-client.json" />
        <property name="hawtio.keycloakServerConfig" value="${jboss.server.config.dir}/keycloak-hawtio.json" />
    </system-properties>
    ```
6.  Add the Hawtio realm to the same file in the `security-domains` section:

    ```xml
    <security-domain name="hawtio" cache-type="default">
        <authentication>
            <login-module code="org.keycloak.adapters.jaas.BearerTokenLoginModule" flag="required">
                <module-option name="keycloak-config-file" value="${hawtio.keycloakServerConfig}"/>
            </login-module>
        </authentication>
    </security-domain>
    ```
7.  Add the `secure-deployment` section `hawtio` to the adapter subsystem. This ensures that the Hawtio WAR is able to find the JAAS login module classes.

    ```xml
    <subsystem xmlns="urn:jboss:domain:keycloak:1.1">
        <secure-deployment name="hawtio-wildfly-1.4.0.redhat-630254.war" />
    </subsystem>
    ```
8.  Restart the Wildfly 10 server with Hawtio:

    ```xml
    cd $EAP_HOME/bin
    ./standalone.sh -Djboss.socket.binding.port-offset=101
    ```
9. Access Hawtio at [http://localhost:8181/hawtio](http://localhost:8181/hawtio). It is secured by Keycloak.
