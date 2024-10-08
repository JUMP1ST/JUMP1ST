# Spring Security Adapter

To secure an application with Spring Security and Keycloak, add this adapter as a dependency to your project. You then have to provide some extra beans in your Spring Security configuration file and add the Keycloak security filter to your pipeline.

Unlike the other Keycloak Adapters, you should not configure your security in web.xml. However, keycloak.json is still required.

**Adapter Installation**

Add Keycloak Spring Security adapter as a dependency to your Maven POM or Gradle build.

```xml
<dependency>
    <groupId>org.keycloak</groupId>
    <artifactId>keycloak-spring-security-adapter</artifactId>
    <version>SNAPSHOT</version>
</dependency>
```

**Spring Security Configuration**

The Keycloak Spring Security adapter takes advantage of Spring Security’s flexible security configuration syntax.

**Java Configuration**

Keycloak provides a KeycloakWebSecurityConfigurerAdapter as a convenient base class for creating a [WebSecurityConfigurer](http://docs.spring.io/spring-security/site/docs/4.0.x/apidocs/org/springframework/security/config/annotation/web/WebSecurityConfigurer.html) instance. The implementation allows customization by overriding methods. While its use is not required, it greatly simplifies your security context configuration.

```
@KeycloakConfiguration
public class SecurityConfig extends KeycloakWebSecurityConfigurerAdapter
{
    /**
     * Registers the KeycloakAuthenticationProvider with the authentication manager.
     */
    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth.authenticationProvider(keycloakAuthenticationProvider());
    }

    /**
     * Defines the session authentication strategy.
     */
    @Bean
    @Override
    protected SessionAuthenticationStrategy sessionAuthenticationStrategy() {
        return new RegisterSessionAuthenticationStrategy(new SessionRegistryImpl());
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception
    {
        super.configure(http);
        http
                .authorizeRequests()
                .antMatchers("/customers*").hasRole("USER")
                .antMatchers("/admin*").hasRole("ADMIN")
                .anyRequest().permitAll();
    }
}
```

You must provide a session authentication strategy bean which should be of type `RegisterSessionAuthenticationStrategy` for public or confidential applications and `NullAuthenticatedSessionStrategy` for bearer-only applications.

Spring Security’s `SessionFixationProtectionStrategy` is currently not supported because it changes the session identifier after login via Keycloak. If the session identifier changes, universal log out will not work because Keycloak is unaware of the new session identifier.

| Tip | The `@KeycloakConfiguration` annotation is a metadata annotion that defines all annotations that are needed to integrate KeyCloak in Spring security. If you have a complexe Spring security setup you can simply have a look ath the annotations of the `@KeycloakConfiguration` annotation and create your own custom meta annotation or just use specific Spring annotations for the KeyCloak adapter. |
| --- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

**XML Configuration**

While Spring Security’s XML namespace simplifies configuration, customizing the configuration can be a bit verbose.

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:security="http://www.springframework.org/schema/security"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/security
       http://www.springframework.org/schema/security/spring-security.xsd">

    <context:component-scan base-package="org.keycloak.adapters.springsecurity" />

    <security:authentication-manager alias="authenticationManager">
        <security:authentication-provider ref="keycloakAuthenticationProvider" />
    </security:authentication-manager>

    <bean id="adapterDeploymentContext" class="org.keycloak.adapters.springsecurity.AdapterDeploymentContextFactoryBean">
        <constructor-arg value="/WEB-INF/keycloak.json" />
    </bean>

    <bean id="keycloakAuthenticationEntryPoint" class="org.keycloak.adapters.springsecurity.authentication.KeycloakAuthenticationEntryPoint" />
    <bean id="keycloakAuthenticationProvider" class="org.keycloak.adapters.springsecurity.authentication.KeycloakAuthenticationProvider" />
    <bean id="keycloakPreAuthActionsFilter" class="org.keycloak.adapters.springsecurity.filter.KeycloakPreAuthActionsFilter" />
    <bean id="keycloakAuthenticationProcessingFilter" class="org.keycloak.adapters.springsecurity.filter.KeycloakAuthenticationProcessingFilter">
        <constructor-arg name="authenticationManager" ref="authenticationManager" />
    </bean>

    <bean id="keycloakLogoutHandler" class="org.keycloak.adapters.springsecurity.authentication.KeycloakLogoutHandler">
        <constructor-arg ref="adapterDeploymentContext" />
    </bean>

    <bean id="logoutFilter" class="org.springframework.security.web.authentication.logout.LogoutFilter">
        <constructor-arg name="logoutSuccessUrl" value="/" />
        <constructor-arg name="handlers">
            <list>
                <ref bean="keycloakLogoutHandler" />
                <bean class="org.springframework.security.web.authentication.logout.SecurityContextLogoutHandler" />
            </list>
        </constructor-arg>
        <property name="logoutRequestMatcher">
            <bean class="org.springframework.security.web.util.matcher.AntPathRequestMatcher">
                <constructor-arg name="pattern" value="/sso/logout**" />
                <constructor-arg name="httpMethod" value="GET" />
            </bean>
        </property>
    </bean>

    <security:http auto-config="false" entry-point-ref="keycloakAuthenticationEntryPoint">
        <security:custom-filter ref="keycloakPreAuthActionsFilter" before="LOGOUT_FILTER" />
        <security:custom-filter ref="keycloakAuthenticationProcessingFilter" before="FORM_LOGIN_FILTER" />
        <security:intercept-url pattern="/customers**" access="ROLE_USER" />
        <security:intercept-url pattern="/admin**" access="ROLE_ADMIN" />
        <security:custom-filter ref="logoutFilter" position="LOGOUT_FILTER" />
    </security:http>

</beans>
```

**Multi Tenancy**

The Keycloak Spring Security adapter also supports multi tenancy. Instead of injecting `AdapterDeploymentContextFactoryBean` with the path to `keycloak.json` you can inject an implementation of the `KeycloakConfigResolver` interface. More details on how to implement the `KeycloakConfigResolver` can be found in [Multi Tenancy](https://wjw465150.gitbooks.io/keycloak-documentation/content/securing\_apps/topics/oidc/java/multi-tenancy.html#\_multi\_tenancy).

**Naming Security Roles**

Spring Security, when using role-based authentication, requires that role names start with `ROLE_`. For example, an administrator role must be declared in Keycloak as `ROLE_ADMIN` or similar, not simply `ADMIN`.

The class `org.keycloak.adapters.springsecurity.authentication.KeycloakAuthenticationProvider` supports an optional `org.springframework.security.core.authority.mapping.GrantedAuthoritiesMapper` which can be used to map roles coming from Keycloak to roles recognized by Spring Security. Use, for example, `org.springframework.security.core.authority.mapping.SimpleAuthorityMapper` to insert the `ROLE_` prefix and convert the role name to upper case. The class is part of Spring Security Core module.

**Client to Client Support**

To simplify communication between clients, Keycloak provides an extension of Spring’s `RestTemplate` that handles bearer token authentication for you. To enable this feature your security configuration must add the `KeycloakRestTemplate` bean. Note that it must be scoped as a prototype to function correctly.

For Java configuration:

```
@Configuration
@EnableWebSecurity
@ComponentScan(basePackageClasses = KeycloakSecurityComponents.class)
public class SecurityConfig extends KeycloakWebSecurityConfigurerAdapter {

    ...

    @Autowired
    public KeycloakClientRequestFactory keycloakClientRequestFactory;

    @Bean
    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    public KeycloakRestTemplate keycloakRestTemplate() {
        return new KeycloakRestTemplate(keycloakClientRequestFactory);
    }

    ...
}
```

For XML configuration:

```
<bean id="keycloakRestTemplate" class="org.keycloak.adapters.springsecurity.client.KeycloakRestTemplate" scope="prototype">
    <constructor-arg name="factory" ref="keycloakClientRequestFactory" />
</bean>
```

Your application code can then use `KeycloakRestTemplate` any time it needs to make a call to another client. For example:

```
@Service
public class RemoteProductService implements ProductService {

    @Autowired
    private KeycloakRestTemplate template;

    private String endpoint;

    @Override
    public List<String> getProducts() {
        ResponseEntity<String[]> response = template.getForEntity(endpoint, String[].class);
        return Arrays.asList(response.getBody());
    }
}
```

**Spring Boot Integration**

The Spring Boot and the Spring Security adapters can be combined.

If you are using the Keycloak Spring Boot Starter to make use of the Spring Security adapter you just need to add the Spring Security starter :

```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

**Using Spring Boot Configuration**

By Default, the Spring Security Adapter looks for a `keycloak.json` configuration file. You can make sure it looks at the configuration provided by the Spring Boot Adapter by adding this bean :

```
@Bean
public KeycloakConfigResolver KeycloakConfigResolver() {
    return new KeycloakSpringBootConfigResolver();
}
```

**Avoid double Filter bean registration**

Spring Boot attempts to eagerly register filter beans with the web application context. Therefore, when running the Keycloak Spring Security adapter in a Spring Boot environment, it may be necessary to add two `FilterRegistrationBean`s to your security configuration to prevent the Keycloak filters from being registered twice.

```
@Configuration
@EnableWebSecurity
public class SecurityConfig extends KeycloakWebSecurityConfigurerAdapter
{
    ...

    @Bean
    public FilterRegistrationBean keycloakAuthenticationProcessingFilterRegistrationBean(
            KeycloakAuthenticationProcessingFilter filter) {
        FilterRegistrationBean registrationBean = new FilterRegistrationBean(filter);
        registrationBean.setEnabled(false);
        return registrationBean;
    }

    @Bean
    public FilterRegistrationBean keycloakPreAuthActionsFilterRegistrationBean(
            KeycloakPreAuthActionsFilter filter) {
        FilterRegistrationBean registrationBean = new FilterRegistrationBean(filter);
        registrationBean.setEnabled(false);
        return registrationBean;
    }

    ...
}
```

\
