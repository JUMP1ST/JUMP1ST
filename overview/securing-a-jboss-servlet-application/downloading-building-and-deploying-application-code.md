# Downloading, Building, and Deploying Application Code

The project and code for the application you are going to secure is available in [Keycloak Quickstarts Repository](https://github.com/keycloak/keycloak-quickstarts). You will need the following installed on your machine and available in your PATH before you can continue:

* Java JDK 8
* Apache Maven 3.1.1 or higher
* Git

You can obtain the code by cloning the repository at [https://github.com/keycloak/keycloak-quickstarts](https://github.com/keycloak/keycloak-quickstarts). Use the branch matching the version of Red Hat Single Sign-On in use. Follow these steps to download the code, build it, and deploy it. Make sure your Wildfly application server is started before you run these steps.

Clone Project

```
$ git clone https://github.com/keycloak/keycloak-quickstarts
$ cd keycloak-quickstarts/app-profile-jee-vanilla
$ mvn clean wildfly:deploy
```

You should see some text scroll down in the application server console window. After the application is successfully deployed go to:

[http://localhost:8080/vanilla](http://localhost:8080/vanilla)

Application Login Page

![app-login-page.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/getting\_started/keycloak-images/app-login-page.png)

If you open up the application’s _web.xml_ file you would see that the application is secured via `BASIC` authentication. If you click on the login button on the login page, the browser will pop up a BASIC auth login dialog.

Application Login Dialog

![client-auth-required.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/getting\_started/keycloak-images/client-auth-required.png)

The application is not secured by any identity provider, so anything you enter in the dialog box will result in a `Forbidden` message being sent back by the server. The next section describes how you can take this deployed application and secure it.
