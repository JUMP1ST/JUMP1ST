# Service Provider Interfaces (SPI)

Keycloak is designed to cover most use-cases without requiring custom code, but we also want it to be customizable. To achieve this Keycloak has a number of Service Provider Interfaces (SPI) for which you can implement your own providers.

#### Implementing a SPI <a href="#implementing_a_spi" id="implementing_a_spi"></a>

To implement an SPI you need to implement its ProviderFactory and Provider interfaces. You also need to create a service configuration file.

For example, to implement the Event Listener SPI you need to implement EventListenerProviderFactory and EventListenerProvider and also provide the file `META-INF/services/org.keycloak.events.EventListenerProviderFactory`.

Example EventListenerProviderFactory:

```java
package org.acme.provider;

import ...

public class MyEventListenerProviderFactory implements EventListenerProviderFactory {

    private List<Event> events;

    public String getId() {
        return "my-event-listener";
    }

    public void init(Config.Scope config) {
        events = new LinkedList();
    }

    public void postInit(KeycloakSessionFactory factory) {
    }

    public EventListenerProvider create(KeycloakSession session) {
        return new MyEventListenerProvider(events);
    }

    public void close() {
    }
}
```

| Note | Keycloak creates a single instance of `EventListenerProviderFactory` which makes it possible to store state for multiple requests. `EventListenerProvider` instances are created by calling create on the factory for each requests so these should be light-weight object. |
| ---- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

Example EventListenerProvider:

```java
package org.acme.provider;

import ...

public class MyEventListenerProvider implements EventListenerProvider {

    private List<Event> events;

    public MyEventListenerProvider(List<Event> events) {
        this.events = events;
    }

    @Override
    public void onEvent(Event event) {
        events.add(event);
    }

    @Override
    public void close() {

    }

    @Override
    public void onEvent(AdminEvent event, boolean includeRepresentation) {
        // Assume this implementation just ignores admin events
    }
}
```

Example service configuration file (`META-INF/services/org.keycloak.events.EventListenerProviderFactory`):

```
org.acme.provider.MyEventListenerProviderFactory
```

You can configure your provider through `standalone.xml`, `standalone-ha.xml`, or `domain.xml`. See the [Server Installation and Configuration](https://keycloak.gitbooks.io/documentation/content/server\_installation/index.html) for more details on where the `standalone.xml`, `standalone-ha.xml`, or `domain.xml` file lives.

For example by adding the following to `standalone.xml`:

```xml
<spi name="eventsListener">
    <provider name="my-event-listener" enabled="true">
        <properties>
            <property name="aNumber" value="10"/>
            <property name="aString" value="Foo"/>
        </properties>
    </provider>
</spi>
```

Then you can retrieve the config in the `ProviderFactory` init method:

```java
public void init(Config.Scope config) {
    Integer aNumber = config.getInt("aNumber");
    String aString = config.get("aString");
}
```

Your provider can also lookup other providers if needed. For example:

```java
public class MyEventListenerProvider implements EventListenerProvider {

    private KeycloakSession session;
    private List<Event> events;

    public MyEventListenerProvider(KeycloakSession session, List<Event> events) {
        this.session = session;
        this.events = events;
    }

    public void onEvent(Event event) {
        RealmModel realm = session.realms().getRealm(event.getRealmId());
        UserModel user = session.users().getUserById(event.getUserId(), realm);

        EmailSenderProvider emailSender = session.getProvider(EmailSenderProvider.class);
        emailSender.send(realm, user, "Hello", "Hello plain text", "<h1>Hello html</h1>" );
    }

    ...

}
```

**Show info from you SPI implementation in Keycloak admin console**

Sometimes it is useful to show additional info about your Provider to a Keycloak administrator. You can show provider build time informations (eg. version of custom provider currently installed), current configuration of the provider (eg. url of remote system your provider talks to) or some operational info (average time of response from remote system your provider talks to). Keycloak admin console provides Server Info page to show this kind of information.

To show info from your provider it is enough to implement `org.keycloak.provider.ServerInfoAwareProviderFactory` interface in your `ProviderFactory`.

Example implementation for `MyEventListenerProviderFactory` from previous example:

```java
package org.acme.provider;

import ...

public class MyEventListenerProviderFactory implements EventListenerProviderFactory, ServerInfoAwareProviderFactory {
    ...

    @Override
    public Map<String, String> getOperationalInfo() {
        Map<String, String> ret = new LinkedHashMap<>();
        ret.put("version", "1.0");
        ret.put("listSizeMax", max + "");
        ret.put("listSizeCurrent", events.size() + "");
        return ret;
    }
}
```

#### Registering provider implementations <a href="#registering_provider_implementations" id="registering_provider_implementations"></a>

There are two ways to register provider implementations. In most cases the simplest way is to use the Keyclopak Deployer approach as this handles a number of dependencies automatically for you. It also supports hot deployment as well as re-deployment.

The alternative approach is to deploy as a module.

If you are creating a custom SPI you will need to deploy it as a module, otherwise we recommend using the Keycloak Deployer approach.

**Using the Keycloak Deployer**

If you copy your provider jar to the Keycloak `deploy/` directory, your provider will automatically be deployed. Hot deployment works too. Additionally, your provider jar works similarly to other components deployed in a JBoss/Wildfly environment in that they can use facilities like the `jboss-deployment-structure.xml` file. This file allows you to set up dependencies on other components and load third-party jars and modules.

Provider jars can also be contained within other deployable units like EARs and WARs. Deploying with a EAR actually makes it really easy to use third party jars as you can just put these libraries in the EAR’s `lib/` directory.

**Register a provider using Modules**

To register a provider using Modules first create a module. To do this you can either use the jboss-cli script or manually create a folder inside `KEYCLOAK_HOME/modules` and add your jar and a `module.xml`. For example to add the event listener sysout example provider using the `jboss-cli` script execute:

```
KEYCLOAK_HOME/bin/jboss-cli.sh --command="module add --name=org.keycloak.examples.event-sysout --resources=target/event-listener-sysout-example.jar --dependencies=org.keycloak.keycloak-core,org.keycloak.keycloak-server-spi,org.keycloak.keycloak-events-api"
```

Or to manually create it start by creating the folder `KEYCLOAK_HOME/modules/org/keycloak/examples/event-sysout/main`. Then copy `event-listener-sysout-example.jar` to this folder and create `module.xml` with the following content:

```
<?xml version="1.0" encoding="UTF-8"?>
<module xmlns="urn:jboss:module:1.3" name="org.keycloak.examples.event-sysout">
    <resources>
        <resource-root path="event-listener-sysout-example.jar"/>
    </resources>
    <dependencies>
        <module name="org.keycloak.keycloak-core"/>
        <module name="org.keycloak.keycloak-server-spi"/>
    </dependencies>
</module>
```

Once you’ve created the module you need to register this module with Keycloak. This is done by editing the keycloak-server subsystem section of `standalone.xml`, `standalone-ha.xml`, or `domain.xml`, and adding it to the providers:

```xml
<subsystem xmlns="urn:jboss:domain:keycloak-server:1.1">
    <web-context>auth</web-context>
    <providers>
        <provider>module:org.keycloak.examples.event-sysout</provider>
    </providers>
    ...
```

**Configuring a provider**

You can pass configuration options to your provider by setting them in `standalone.xml`, `standalone-ha.xml`, or `domain.xml`. For example to set the max value for `my-event-listener` add:

```
<spi name="eventsListener">
    <provider name="my-event-listener" enabled="true">
        <properties>
            <property name="max" value="100"/>
        </properties>
    </provider>
</spi>
```

**Disabling a provider**

You can disable a provider by setting the enabled attribute for the provider to false in `standalone.xml`, `standalone-ha.xml`, or `domain.xml`. For example to disable the Infinispan user cache provider add:

```xml
<spi name="userCache">
    <provider name="infinispan" enabled="false"/>
</spi>
```

#### Leveraging Java EE <a href="#leveraging_java_ee" id="leveraging_java_ee"></a>

The can be packaged within any Java EE component so long as you set up the `META-INF/services` file correctly to point to your providers. For example, if your provider needs to use third party libraries, you can package up your provider within an ear and store these third pary libraries in the ear’s `lib/` directory. Also note that provider jars can make use of the `jboss-deployment-structure.xml` file that EJBs, WARS, and EARs can use in a JBoss/Wildfly environment. See the JBoss/Wildfly documentation for more details on this file. It allows you to pull in external dependencies among other fine grain actions.

`ProviderFactory` implementations are required to be plain java objects. But, we also currently support implementing provider classes as Stateful EJBs. TThis is how you would do it:

```java
@Stateful
@Local(EjbExampleUserStorageProvider.class)
public class EjbExampleUserStorageProvider implements UserStorageProvider,
        UserLookupProvider,
        UserRegistrationProvider,
        UserQueryProvider,
        CredentialInputUpdater,
        CredentialInputValidator,
        OnUserCache
{
    @PersistenceContext
    protected EntityManager em;

    protected ComponentModel model;
    protected KeycloakSession session;

    public void setModel(ComponentModel model) {
        this.model = model;
    }

    public void setSession(KeycloakSession session) {
        this.session = session;
    }


    @Remove
    @Override
    public void close() {
    }
...
}
```

You have to define the `@Local` annotation and specify your provider class there. If you don’t do this, EJB will not proxy the provider instance correctly and your provider won’t work.

You must put the `@Remove` annotation on the `close()` method of your provider. If you don’t, the stateful bean will never be cleaned up and you may eventually see error messages.

Implementations of `ProviderFactory` are required to be plain java objects. Your factory class would perform a JNDI lookup of the Stateful EJB in its create() method.

```java
public class EjbExampleUserStorageProviderFactory
        implements UserStorageProviderFactory<EjbExampleUserStorageProvider> {

    @Override
    public EjbExampleUserStorageProvider create(KeycloakSession session, ComponentModel model) {
        try {
            InitialContext ctx = new InitialContext();
            EjbExampleUserStorageProvider provider = (EjbExampleUserStorageProvider)ctx.lookup(
                     "java:global/user-storage-jpa-example/" + EjbExampleUserStorageProvider.class.getSimpleName());
            provider.setModel(model);
            provider.setSession(session);
            return provider;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
```

#### Available SPIs <a href="#available_spis" id="available_spis"></a>

Here’s a list of the most important available SPIs and a brief description. For more details on each SPI refer to individual sections. If you want to see list of all available SPIs at runtime, you can check `Server Info` page in admin console as described in [Admin Console](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_development/topics/providers.html#\_providers\_admin\_console) section.

| SPI                    | Description                                                                                                                                                                                                                                  |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Connections Infinispan | Loads and configures Infinispan connections. The default implementation can load connections from the Infinispan subsystem, or alternatively can be manually configured in standalone.xml                                                    |
| Connections Jpa        | Loads and configures Jpa connections. The default implementation can load datasources from WildFly/EAP, or alternatively can be manually configured in standalone.xml                                                                        |
| Connections Mongo      | Loads and configures MongoDB connections. The default implementation is configured in standalone.xml                                                                                                                                         |
| Email Sender           | Sends email. The default implementation uses JavaMail                                                                                                                                                                                        |
| Email Template         | Format email and uses Email Sender to send the email. The default implementation uses FreeMarker templates                                                                                                                                   |
| Events Listener        | Listen to user related events for example user login success and failures. Keycloak provides two implementations out of box. One that logs events to the server log and another that can send email notifications to users on certain events |
| Login Protocol         | Provides protocols. Keycloak provides implementations of OpenID Connect and SAML 2.0                                                                                                                                                         |
| Realm                  | Provides realm and application meta-data. Keycloak provides implementations for Relational Databases and MongoDB                                                                                                                             |
| Realm Cache            | Caches realm and application meta-data to improve performance. Default implementation uses Infinispan                                                                                                                                        |
| Timer                  | Executes scheduled tasks. Keycloak provides a basic implementation based on java.util.Timer                                                                                                                                                  |
| User                   | Provides users and role-mappings. Keycloak provides implementations for Relational Databases and MongoDB                                                                                                                                     |
| User Cache             | Caches users to improve performance. Default implementation uses Infinispan                                                                                                                                                                  |
| User Federation        | Support syncing users from an external source. Keycloak provides implementations for LDAP and Active Directory                                                                                                                               |
| User Sessions          | Provides users session information. Keycloak provides implementations for basic in-memory, Infinispan, Relational Databases and MongoDB                                                                                                      |
