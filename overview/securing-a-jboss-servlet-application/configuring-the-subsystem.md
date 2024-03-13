# Configuring the Subsystem

Now that you have copied the XML template from the Installation page, you need to paste this into the _standalone.xml_ file that resides in the _standalone/configuration_ directory of the application server instance on which your application is deployed.

1.  Open the standalone/configuration/standalone.xml file and search for the following text:

    ```xml
    <subsystem xmlns="urn:jboss:domain:keycloak:1.1"/>
    ```
2.  Modify this to prepare it for pasting in your template from the Installation page:

    ```xml
    <subsystem xmlns="urn:jboss:domain:keycloak:1.1">
    </subsystem>
    ```
3.  Within the \<subsystem> element, paste in the template. It will look something like this:

    ```xml
    <subsystem xmlns="urn:jboss:domain:keycloak:1.1">
      <secure-deployment name="WAR MODULE NAME.war">
        <realm>demo</realm>
        <auth-server-url>http://localhost:8180/auth</auth-server-url>
        <public-client>true</public-client>
        <ssl-required>EXTERNAL</ssl-required>
        <resource>vanilla</resource>
      </secure-deployment>
    </subsystem>
    ```
4.  Change the **WAR MODULE NAME** text to **vanilla** as follows:

    ```xml
    <subsystem xmlns="urn:jboss:domain:keycloak:1.1">
      <secure-deployment name="vanilla.war">
      ...
    </subsystem>
    ```
5. Reboot your application server.
6. Go to [http://localhost:8080/vanilla](http://localhost:8080/vanilla) and click **login**. The Keycloak login page opens. You can log in using the user you created in the [Creating a New User](https://wjw465150.gitbooks.io/keycloak-documentation/content/getting\_started/topics/first-realm/user.html#\_create-new-user) chapter.
