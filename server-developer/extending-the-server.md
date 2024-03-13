# Extending the Server

The Keycloak SPI framework offers the possibility to implement or override particular built-in providers. However Keycloak also provides capabilities to extend it’s core functionalities and domain. This includes possibilities to:

* Add custom REST endpoints to the Keycloak server
* Add your own custom SPI
* Add custom JPA entities to the Keycloak data model

#### Add custom REST endpoints <a href="#extensions_rest" id="extensions_rest"></a>

This is a very powerful extension, which allows you to deploy your own REST endpoints to the Keycloak server. It enables all kinds of extensions, for example the possibility to trigger functionality on the Keycloak server, which is not available through the default set of built-in Keycloak REST endpoints.

To add a custom REST endpoint, you need to implement the `RealmResourceProviderFactory` and `RealmResourceProvider` interfaces. `RealmResourceProvider` has one important method:

```java
Object getResource();
```

which allows you to return an object, which acts as a [JAX-RS Resource](https://jax-rs-spec.java.net/). For more details, see the Javadoc and our examples. There is a very simple example in the example distribution in `providers/rest` and there is a more advanced example in `providers/domain-extension`, which shows how to add an authenticated REST endpoint and other functionalities like [Adding your own SPI](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_development/topics/extensions.html#\_extensions\_spi) or [Extending datamodel with your own JPA entities](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_development/topics/extensions.html#\_extensions\_jpa).

For details on how to package and deploy a custom provider, refer to the [Service Provider Interfaces](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_development/topics/providers.html#\_providers) chapter.

#### Add your own custom SPI <a href="#extensions_spi" id="extensions_spi"></a>

This is useful especially with the [Custom REST endpoints](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_development/topics/extensions.html#\_extensions\_rest). To add your own kind of SPI, you need to implement the interface `org.keycloak.provider.Spi` and define the ID of your SPI and the `ProviderFactory` and `Provider` classes. That looks like this:

```java
package org.keycloak.examples.domainextension.spi;

import ...

public class ExampleSpi implements Spi {

    @Override
    public boolean isInternal() {
        return false;
    }

    @Override
    public String getName() {
        return "example";
    }

    @Override
    public Class<? extends Provider> getProviderClass() {
        return ExampleService.class;
    }

    @Override
    @SuppressWarnings("rawtypes")
    public Class<? extends ProviderFactory> getProviderFactoryClass() {
        return ExampleServiceProviderFactory.class;
    }

}
```

Then you need to create the file `META-INF/services/org.keycloak.provider.Spi` and add the class of your SPI to it. For example:

```
org.keycloak.examples.domainextension.spi.ExampleSpi
```

The next step is to create the interfaces `ExampleServiceProviderFactory`, which extends from `ProviderFactory` and `ExampleService`, which extends from `Provider`. The `ExampleService` will usually contain the business methods you need for your use case. Note that the `ExampleServiceProviderFactory` instance is always scoped per application, however `ExampleService` is scoped per-request (or more accurately per `KeycloakSession` lifecycle).

Finally you need to implement your providers in the same manner as described in the [Service Provider Interfaces](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_development/topics/providers.html#\_providers) chapter.

For more details, take a look at the example distribution at `providers/domain-extension`, which shows an Example SPI similar to the one above.

#### Add custom JPA entities to the Keycloak data model <a href="#extensions_jpa" id="extensions_jpa"></a>

If the Keycloak data model does not exactly match your desired solution, or if you want to add some core functionality to Keycloak, or when you have your own REST endpoint, you might want to extend the Keycloak data model. We enable you to add your own JPA entities to the Keycloak JPA `EntityManager` .

To add your own JPA entities, you need to implement `JpaEntityProviderFactory` and `JpaEntityProvider`. The `JpaEntityProvider` allows you to return a list of your custom JPA entities and provide the location and id of the liquibase changelog. An example implementation can look like this:

```java
public class ExampleJpaEntityProvider implements JpaEntityProvider {

    // List of your JPA entities.
    @Override
    public List<Class<?>> getEntities() {
        return Collections.<Class<?>>singletonList(Company.class);
    }

    // This is used to return the location of the Liquibase changelog file.
    // You can return null if you don't want Liquibase to create and update the DB schema.
    @Override
    public String getChangelogLocation() {
    	return "META-INF/example-changelog.xml";
    }

    // Helper method, which will be used internally by Liquibase.
    @Override
    public String getFactoryId() {
        return "sample";
    }

    ...
}
```

In the example above, we added a single JPA entity represented by class `Company`. In the code of your REST endpoint, you can then use something like this to retrieve `EntityManager` and call DB operations on it.

```java
EntityManager em = session.getProvider(JpaConnectionProvider.class).getEntityManager();
Company myCompany = em.find(Company.class, "123");
```

The methods `getChangelogLocation` and `getFactoryId` are important to support automatic updating of your entities by Liquibase. [Liquibase](http://www.liquibase.org/) is a framework for updating the database schema, which Keycloak internally uses to create the DB schema and update the DB schema among versions. You may need to use it as well and create a changelog for your entities. Note that versioning of your own liquibase changelog is independent of Keycloak versions. In other words, when you update to a new Keycloak version, you are not forced to update your schema at the same time. And vice versa, you can update your schema even without updating the Keycloak version. The Liquibase update is always done at the server startup, so to trigger a DB update of your schema, you just need to add the new changeset to your liquibase changelog file (in the example above it’s the file `META-INF/example-changelog.xml` (which must be packed in same JAR as the JPA entities and `ExampleJpaEntityProvider`) and then restart server. The DB schema will be automatically updated at startup.

For more details, take a look at the example distribution at example `providers/domain-extension`, which shows the `ExampleJpaEntityProvider` and `example-changelog.xml` described above.

| Note | Don’t forget to always backup your database before doing any changes in the Liquibase changelog and triggering a DB update. |
| ---- | --------------------------------------------------------------------------------------------------------------------------- |

\
