# JBoss EAP/Wildfly Adapter

To be able to secure WAR apps deployed on JBoss EAP or Wildfly, you must install and configure the Keycloak SAML Adapter Subsystem.

You then provide a keycloak config, `/WEB-INF/keycloak-saml.xml` file in your WAR and change the auth-method to KEYCLOAK-SAML within web.xml. Both methods are described in this section.
