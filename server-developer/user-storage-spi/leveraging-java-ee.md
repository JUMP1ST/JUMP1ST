# Leveraging Java EE

The user storage providers can be packaged within any Java EE component if you set up the `META-INF/services` file correctly to point to your providers. For example, if your provider needs to use third-party libraries, you can package up your provider within an EAR and store these third-party libraries in the `lib/` directory of the EAR. Also note that provider JARs can make use of the `jboss-deployment-structure.xml` file that EJBs, WARS, and EARs can use in a JBoss/Wildfly environment. For more details on this file, see the JBoss/Wildfly documentation. It allows you to pull in external dependencies among other fine-grained actions.

Implementations of `UserStorageProviderFactory` are required to be plain java objects. But we also currently support implementing `UserStorageProvider` classes as Stateful EJBs. This is especially useful if you want to use JPA to connect to a relational store. This is how you would do it:

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

You have to define the `@Local` annotation and specify your provider class there. If you do not do this, EJB will not proxy the user correctly and your provider won’t work.

You must put the `@Remove` annotation on the `close()` method of your provider. If you do not, the stateful bean will never be cleaned up and you might eventually see error messages.

Implementations of `UserStorageProviderFactory` are required to be plain java objects. Your factory class would perform a JNDI lookup of the Stateful EJB in its create() method.

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

This example also assumes that you have defined a JPA deployment in the same JAR as the provider. This means a `persistence.xml` file as well as any JPA `@Entity` classes.

| Warning | When using JPA any additional datasource must be an XA datasource. The Keycloak datasource is not an XA datasource. If you interact with two or more non-XA datasources in the same transaction, the server returns an error message. Only one non-XA resource is permitted in a single transaction. |
| ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

See the JBoss/Wildfly manual for more details on deploying an XA datasource.

\
