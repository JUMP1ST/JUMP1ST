# Adapter Installation

Each adapter is a separate download on the Keycloak download site.

Install on Wildfly 9 or 10, 11 or JBoss EAP 7:

```
$ cd $WILDFLY_HOME
$ unzip keycloak-saml-wildfly-adapter-dist.zip
```

Install on JBoss EAP 6.x:

```
$ cd $JBOSS_HOME
$ unzip keycloak-saml-eap6-adapter-dist.zip
```

These zip files create new JBoss Modules specific to the Wildfly/JBoss EAP SAML Adapter within your Wildfly or JBoss EAP distro.

After adding the modules, you must then enable the Keycloak SAML Subsystem within your app server’s server configuration: `domain.xml` or `standalone.xml`.

There is a CLI script that will help you modify your server configuration. Start the server and run the script from the server’s bin directory:

Wildfly 11

```
$ ./bin/jboss-cli.sh --file=adapter-elytron-install-saml.cli
```

Any other server but Wildfly 11

```
$ cd $JBOSS_HOME/bin
$ jboss-cli.sh -c --file=adapter-install-saml.cli
```

The script will add the extension, subsystem, and optional security-domain as described below.

```xml
<server xmlns="urn:jboss:domain:1.4">

    <extensions>
        <extension module="org.keycloak.keycloak-saml-adapter-subsystem"/>
          ...
    </extensions>

    <profile>
        <subsystem xmlns="urn:jboss:domain:keycloak-saml:1.1"/>
         ...
    </profile>
```

The `keycloak` security domain should be used with EJBs and other components when you need the security context created in the secured web tier to be propagated to the EJBs (other EE component) you are invoking. Otherwise this configuration is optional.

```xml
<server xmlns="urn:jboss:domain:1.4">
 <subsystem xmlns="urn:jboss:domain:security:1.2">
    <security-domains>
...
      <security-domain name="keycloak">
         <authentication>
           <login-module code="org.keycloak.adapters.jboss.KeycloakLoginModule"
                         flag="required"/>
          </authentication>
      </security-domain>
    </security-domains>
```

For example, if you have a JAX-RS service that is an EJB within your WEB-INF/classes directory, you’ll want to annotate it with the `@SecurityDomain` annotation as follows:

```xml
import org.jboss.ejb3.annotation.SecurityDomain;
import org.jboss.resteasy.annotations.cache.NoCache;

import javax.annotation.security.RolesAllowed;
import javax.ejb.EJB;
import javax.ejb.Stateless;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import java.util.ArrayList;
import java.util.List;

@Path("customers")
@Stateless
@SecurityDomain("keycloak")
public class CustomerService {

    @EJB
    CustomerDB db;

    @GET
    @Produces("application/json")
    @NoCache
    @RolesAllowed("db_user")
    public List<String> getCustomers() {
        return db.getCustomers();
    }
}
```

We hope to improve our integration in the future so that you don’t have to specify the `@SecurityDomain` annotation when you want to propagate a keycloak security context to the EJB tier.

\
