# Authentication SPI

Keycloak comes out of the box with a bunch of different authentication mechanisms: kerberos, password, and otp. These mechanisms may not meet all of your requirements and you may want to plug in your own custom ones. Keycloak provides an authentication SPI that you can use to write new plugins. The admin console supports applying, ordering, and configuring these new mechanisms.

Keycloak also supports a simple registration form. Different aspects of this form can be enabled and disabled i.e. Recaptcha support can be turned off and on. The same authentication SPI can be used to add another page to the registration flow or reimplement it entirely. There’s also an additional fine-grain SPI you can use to add specific validations and user extensions to the built in registration form.

A required action in Keycloak is an action that a user has to perform after he authenticates. After the action is performed successfully, the user doesn’t have to perform the action again. Keycloak comes with some built in required actions like "reset password". This action forces the user to change their password after they have logged in. You can write and plug in your own required actions.

#### Terms <a href="#terms" id="terms"></a>

To first learn about the Authentication SPI, let’s go over some of the terms used to describe it.

Authentication Flow

A flow is a container for all authentications that must happen during login or registration. If you go to the admin console authentication page, you can view all the defined flows in the system and what authenticators they are made up of. Flows can contain other flows. You can also bind a new different flow for browser login, direct grant access, and registration.

Authenticator

An authenticator is a pluggable component that hold the logic for performing the authentication or action within a flow. It is usually a singleton.

Execution

An execution is an object that binds the authenticator to the flow and the authenticator to the configuration of the authenticator. Flows contain execution entries.

Execution Requirement

Each execution defines how an authenticator behaves in a flow. The requirement defines whether the authenticator is enabled, disabled, optional, required, or an alternative. An alternative requirement means that the authentiactor is optional unless no other alternative authenticator is successful in the flow. For example, cookie authentication, kerberos, and the set of all login forms are all alternative. If one of those is successful, none of the others are executed.

Authenticator Config

This object defines the configuration for the Authenticator for a specific execution within an authentication flow. Each execution can have a different config.

Required Action

After authentication completes, the user might have one or more one-time actions he must complete before he is allowed to login. The user might be required to set up an OTP token generator or reset an expired password or even accept a Terms and Conditions document.

#### Algorithm Overview <a href="#algorithm_overview" id="algorithm_overview"></a>

Let’s talk about how this all works for browser login. Let’s assume the following flows, executions and sub flows.

```
Cookie - ALTERNATIVE
Kerberos - ALTERNATIVE
Forms Subflow - ALTERNATIVE
           Username/Password Form - REQUIRED
           OTP Password Form - OPTIONAL
```

In the top level of the form we have 3 executions of which all are alternatively required. This means that if any of these are successful, then the others do not have to execute. The Username/Password form is not executed if there is an SSO Cookie set or a successful Kerberos login. Let’s walk through the steps from when a client first redirects to keycloak to authenticate the user.

1. The OpenID Connect or SAML protocol provider unpacks relevent data, verifies the client and any signatures. It creates a AuthenticationSessionModel. It looks up what the browser flow should be, then starts executing the flow.
2. The flow looks at the cookie execution and sees that it is an alternative. It loads the cookie provider. It checks to see if the cookie provider requires that a user already be associated with the authentication session. Cookie provider does not require a user. If it did, the flow would abort and the user would see an error screen. Cookie provider then executes. Its purpose is to see if there is an SSO cookie set. If there is one set, it is validated and the UserSessionModel is verified and associated with the AuthenticationSessionModel. The Cookie provider returns a success() status if the SSO cookie exists and is validated. Since the cookie provider returned success and each execution at this level of the flow is ALTERNATIVE, no other execution is executed and this results in a successful login. If there is no SSO cookie, the cookie provider returns with a status of attempted(). This means there was no error condition, but no success either. The provider tried, but the request just wasn’t set up to handle this authenticator.
3. Next the flow looks at the Kerberos execution. This is also an alternative. The kerberos provider also does not require a user to be already set up and associated with the AuthenticationSessionModel so this provider is executed. Kerberos uses the SPNEGO browser protocol. This requires a series of challenge/responses between the server and client exchanging negotiation headers. The kerberos provider does not see any negotiate header, so it assumes that this is the first interaction between the server and client. It therefore creates an HTTP challenge response to the client and sets a forceChallenge() status. A forceChallenge() means that this HTTP response cannot be ignored by the flow and must be returned to the client. If instead the provider returned a challenge() status, the flow would hold the challenge response until all other alternatives are attempted. So, in this initial phase, the flow would stop and the challenge response would be sent back to the browser. If the browser then responds with a successful negotiate header, the provider associates the user with the AuthenticationSession and the flow ends because the rest of the executions on this level of the flow are all alternatives. Otherwise, again, the kerberos provider sets an attempted() status and the flow continues.
4. The next execution is a subflow called Forms. The executions for this subflow are loaded and the same processing logic occurs
5. The first execution in the Forms subflow is the UsernamePassword provider. This provider also does not require for a user to already be associated with the flow. This provider creates challenge HTTP response and sets its status to challenge(). This execution is required, so the flow honors this challenge and sends the HTTP response back to the browser. This response is a rendering of the Username/Password HTML page. The user enters in their username and password and clicks submit. This HTTP request is directed to the UsernamePassword provider. If the user entered an invalid username or password, a new challenge response is created and a status of failureChallenge() is set for this execution. A failureChallenge() means that there is a challenge, but that the flow should log this as an error in the error log. This error log can be used to lock accounts or IP Addresses that have had too many login failures. If the username and password is valid, the provider associated the UserModel with the AuthenticationSessionModel and returns a status of success()
6. The next execution is the OTP Form. This provider requires that a user has been associated with the flow. This requirement is satisfied because the UsernamePassword provider already associated the user with the flow. Since a user is required for this provider, the provider is also asked if the user is configured to use this provider. If user is not configured, and this execution is required, then the flow will then set up a required action that the user must perform after authentication is complete. For OTP, this means the OTP setup page. If the execution was optional, then this execution is skipped.
7. After the flow is complete, the authentication processor creates a UserSessionModel and associates it with the AuthenticationSessionModel. It then checks to see if the user is required to complete any required actions before logging in.
8. First, each required action’s evaluateTriggers() method is called. This allows the required action provider to figure out if there is some state that might trigger the action to be fired. For example, if your realm has a password expiration policy, it might be triggered by this method.
9. Each required action associated with the user that has its requiredActionChallenge() method called. Here the provider sets up an HTTP response which renders the page for the required action. This is done by setting a challenge status.
10. If the required action is ultimately successful, then the required action is removed from the user’s require actions list.
11. After all required actions have been resolved, the user is finally logged in.

#### Authenticator SPI Walk Through <a href="#auth_spi_walkthrough" id="auth_spi_walkthrough"></a>

In this section, we’ll take a look at the Authenticator interface. For this, we are going to implement an authenticator that requires that a user enter in the answer to a secret question like "What is your mother’s maiden name?". This example is fully implemented and contained in the examples/providers/authenticator directory of the demo distribution of Keycloak.

The classes you must implement are the org.keycloak.authentication.AuthenticatorFactory and Authenticator interfaces. The Authenticator interface defines the logic. The AuthenticatorFactory is responsible for creating instances of an Authenticator. They both extend a more generic Provider and ProviderFactory set of interfaces that other Keycloak components like User Federation do.

**Packaging Classes and Deployment**

You will package your classes within a single jar. This jar must contain a file named `org.keycloak.authentication.AuthenticatorFactory` and must be contained in the `META-INF/services/` directory of your jar. This file must list the fully qualified classname of each AuthenticatorFactory implementation you have in the jar. For example:

```
org.keycloak.examples.authenticator.SecretQuestionAuthenticatorFactory
org.keycloak.examples.authenticator.AnotherProviderFactory
```

This services/ file is used by Keycloak to scan the providers it has to load into the system.

To deploy this jar, just copy it to the providers directory.

**Implementing an Authenticator**

When implementing the Authenticator interface, the first method that needs to be implemented is the requiresUser() method. For our example, this method must return true as we need to validate the secret question associated with the user. A provider like kerberos would return false from this method as it can resolve a user from the negotiate header. This example, however, is validating a specific credential of a specific user.

The next method to implement is the configuredFor() method. This method is responsible for determining if the user is configured for this particular authenticator. For this example, we need to check if the answer to the secret question has been set up by the user or not. In our case we are storing this information, hashed, within a UserCredentialValueModel within the UserModel (just like passwords are stored). Here’s how we do this very simple check:

```java
@Override
  public boolean configuredFor(KeycloakSession session, RealmModel realm, UserModel user) {
     return session.users().configuredForCredentialType("secret_question", realm, user);
    }
```

The configuredForCredentialType() call queries the user to see if it supports that credential type.

The next method to implement on the Authenticator is setRequiredActions(). If configuredFor() returns false and our example authenticator is required within the flow, this method will be called. It is responsible for registering any required actions that must be performed by the user. In our example, we need to register a required action that will force the user to set up the answer to the secret question. We will implement this required action provider later in this chapter. Here is the implementation of the setRequiredActions() method.

```java
    @Override
    public void setRequiredActions(KeycloakSession session, RealmModel realm, UserModel user) {
        user.addRequiredAction("SECRET_QUESTION_CONFIG");
    }
```

Now we are getting into the meat of the Authenticator implementation. The next method to implement is authenticate(). This is the initial method the flow invokes when the execution is first visited. What we want is that if a user has answered the secret question already on their browser’s machine, then the user doesn’t have to answer the question again, making that machine "trusted". The authenticate() method isn’t responsible for processing the secret question form. Its sole purpose is to render the page or to continue the flow.

```java
    @Override
    public void authenticate(AuthenticationFlowContext context) {
        if (hasCookie(context)) {
           context.success();
           return;
        }
        Response challenge = loginForm(context).createForm("secret_question.ftl");
        context.challenge(challenge);
    }
```

The hasCookie() method checks to see if there is already a cookie set on the browser which indicates that the secret question has already been answered. If that returns true, we just mark this execution’s status as SUCCESS using the AuthenticationFlowContext.success() method and returning from the authentication() method.

If the hasCookie() method returns false, we must return a response that renders the secret question HTML form. AuthenticationFlowContext has a form() method that initializes a Freemarker page builder with appropriate base information needed to build the form. This page builder is called `org.keycloak.login.LoginFormsProvider`. the LoginFormsProvider.createForm() method loads a Freemarker template file from your login theme. Additionally you can call the LoginFormsProvider.setAttribute() method if you want to pass additional information to the Freemarker template. We’ll go over this later.

Calling LoginFormsProvider.createForm() returns a JAX-RS Response object. We then call AuthenticationFlowContext.challenge() passing in this response. This sets the status of the execution as CHALLENGE and if the execution is Required, this JAX-RS Response object will be sent to the browser.

So, the HTML page asking for the answer to a secret question is displayed to the user and the user enteres in the answer and clicks submit. The action URL of the HTML form will send an HTTP request to the flow. The flow will end up invoking the action() method of our Authenticator implementation.

```java
    @Override
    public void action(AuthenticationFlowContext context) {
        boolean validated = validateAnswer(context);
        if (!validated) {
           Response challenge = context.form()
                                 .setError("badSecret")
                                 .createForm("secret-question.ftl");
           context.failureChallenge(AuthenticationFlowError.INVALID_CREDENTIALS, challenge);
           return;
        }
        setCookie(context);
        context.success();
    }
```

If the answer is not valid, we rebuild the HTML Form with an additional error message. We then call AuthenticationFlowContext.failureChallenge() passing in the reason for the value and the JAX-RS response. failureChallenge() works the same as challenge(), but it also records the failure so it can be analyzed by any attack detection service.

If validation is successful, then we set a cookie to remember that the secret question has been answered and we call AuthenticationFlowContext.success().

The last thing I want to go over is the setCookie() method. This is an example of providing configuration for the Authenticator. In this case we want the max age of the cookie to be configurable.

```java
    protected void setCookie(AuthenticationFlowContext context) {
        AuthenticatorConfigModel config = context.getAuthenticatorConfig();
        int maxCookieAge = 60 * 60 * 24 * 30; // 30 days
        if (config != null) {
            maxCookieAge = Integer.valueOf(config.getConfig().get("cookie.max.age"));

        }
        ... set the cookie ...
    }
```

We obtain an AuthenticatorConfigModel from the AuthenticationFlowContext.getAuthenticatorConfig() method. If configuration exists we pull the max age config out of it. We will see how we can define what should be configured when we talk about the AuthenticatorFactory implementation. The config values can be defined within the admin console if you set up config definitions in your AuthenticatorFactory implementation.

**Implementing an AuthenticatorFactory**

The next step in this process is to implement an AuthenticatorFactory. This factory is responsible for instantiating an Authenticator. It also provides deployment and configuration metadata about the Authenticator.

The getId() method is just the unique name of the component. The create() method is called by the runtime to allocate and process the Authenticator.

```java
public class SecretQuestionAuthenticatorFactory implements AuthenticatorFactory, ConfigurableAuthenticatorFactory {

    public static final String PROVIDER_ID = "secret-question-authenticator";
    private static final SecretQuestionAuthenticator SINGLETON = new SecretQuestionAuthenticator();

    @Override
    public String getId() {
        return PROVIDER_ID;
    }

    @Override
    public Authenticator create(KeycloakSession session) {
        return SINGLETON;
    }
```

The next thing the factory is responsible for is specify the allowed requirement switches. While there are four different requirement types: ALTERNATIVE, REQUIRED, OPTIONAL, DISABLED, AuthenticatorFactory implementations can limit which requirement options are shown in the admin console when defining a flow. In our example, we’re going to limit our requirement options to REQUIRED and DISABLED.

```java
    private static AuthenticationExecutionModel.Requirement[] REQUIREMENT_CHOICES = {
            AuthenticationExecutionModel.Requirement.REQUIRED,
            AuthenticationExecutionModel.Requirement.DISABLED
    };
    @Override
    public AuthenticationExecutionModel.Requirement[] getRequirementChoices() {
        return REQUIREMENT_CHOICES;
    }
```

The AuthenticatorFactory.isUserSetupAllowed() is a flag that tells the flow manager whether or not Authenticator.setRequiredActions() method will be called. If an Authenticator is not configured for a user, the flow manager checks isUserSetupAllowed(). If it is false, then the flow aborts with an error. If it returns true, then the flow manager will invoke Authenticator.setRequiredActions().

```java
    @Override
    public boolean isUserSetupAllowed() {
        return true;
    }
```

The next few methods define how the Authenticator can be configured. The isConfigurable() method is a flag which specifies to the admin console on whether the Authenticator can be configured within a flow. The getConfigProperties() method returns a list of ProviderConfigProperty objects. These objects define a specific configuration attribute.

```java
    @Override
    public List<ProviderConfigProperty> getConfigProperties() {
        return configProperties;
    }

    private static final List<ProviderConfigProperty> configProperties = new ArrayList<ProviderConfigProperty>();

    static {
        ProviderConfigProperty property;
        property = new ProviderConfigProperty();
        property.setName("cookie.max.age");
        property.setLabel("Cookie Max Age");
        property.setType(ProviderConfigProperty.STRING_TYPE);
        property.setHelpText("Max age in seconds of the SECRET_QUESTION_COOKIE.");
        configProperties.add(property);
    }
```

Each ProviderConfigProperty defines the name of the config property. This is the key used in the config map stored in AuthenticatorConfigModel. The label defines how the config option will be displayed in the admin console. The type defines if it is a String, Boolean, or other type. The admin console will display different UI inputs depending on the type. The help text is what will be shown in the tooltip for the config attribute in the admin console. Read the javadoc of ProviderConfigProperty for more detail.

The rest of the methods are for the admin console. getHelpText() is the tooltip text that will be shown when you are picking the Authenticator you want to bind to an execution. getDisplayType() is what text that will be shown in the admin console when listing the Authenticator. getReferenceCategory() is just a category the Authenticator belongs to.

**Adding Authenticator Form**

Keycloak comes with a Freemarker [theme and template engine](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_development/topics/themes.html#\_themes). The createForm() method you called within authenticate() of your Authenticator class, builds an HTML page from a file within your login theme: secret-question.ftl. This file should be placed in the login theme with all the other .ftl files you see for login.

Let’s take a bigger look at secret-question.ftl Here’s a small code snippet:

```java
        <form id="kc-totp-login-form" class="${properties.kcFormClass!}" action="${url.loginAction}" method="post">
            <div class="${properties.kcFormGroupClass!}">
                <div class="${properties.kcLabelWrapperClass!}">
                    <label for="totp" class="${properties.kcLabelClass!}">${msg("loginSecretQuestion")}</label>
                </div>

                <div class="${properties.kcInputWrapperClass!}">
                    <input id="totp" name="secret_answer" type="text" class="${properties.kcInputClass!}" />
                </div>
            </div>
```

Any piece of text enclosed in `${}` corresponds to an attribute or template funtion. If you see the form’s action, you see it points to `${url.loginAction}`. This value is automatically generated when you invoke the AuthenticationFlowContext.form() method. You can also obtain this value by calling the AuthenticationFlowContext.getActionURL() method in Java code.

You’ll also see `${properties.someValue}`. These correspond to properties defined in your theme.properties file of our theme. `${msg("someValue")}` corresponds to the internationalized message bundles (.properties files) included with the login theme messages/ directory. If you’re just using english, you can just add the value of the `loginSecretQuestion` value. This should be the question you want to ask the user.

When you call AuthenticationFlowContext.form() this gives you a LoginFormsProvider instance. If you called, `LoginFormsProvider.setAttribute("foo", "bar")`, the value of "foo" would be available for reference in your form as `${foo}`. The value of an attribute can be any Java bean as well.

**Adding Authenticator to a Flow**

Adding an Authenticator to a flow must be done in the admin console. If you go to the Authentication menu item and go to the Flow tab, you will be able to view the currently defined flows. You cannot modify an built in flows, so, to add the Authenticator we’ve created you have to copy an existing flow or create your own. I’m hoping the UI is intuitive enough so that you can figure out for yourself how to create a flow and add the Authenticator.

After you’ve created your flow, you have to bind it to the login action you want to bind it to. If you go to the Authentication menu and go to the Bindings tab you will see options to bind a flow to the browser, registration, or direct grant flow.

#### Required Action Walkthrough <a href="#required_action_walkthrough" id="required_action_walkthrough"></a>

In this section we will discuss how to define a required action. In the Authenticator section you may have wondered, "How will we get the user’s answer to the secret question entered into the system?". As we showed in the example, if the answer is not set up, a required action will be triggered. This section discusses how to implement the required action for the Secret Question Authenticator.

**Packaging Classes and Deployment**

You will package your classes within a single jar. This jar does not have to be separate from other provider classes but it must contain a file named `org.keycloak.authentication.RequiredActionFactory` and must be contained in the `META-INF/services/` directory of your jar. This file must list the fully qualified classname of each RequiredActionFactory implementation you have in the jar. For example:

```java
org.keycloak.examples.authenticator.SecretQuestionRequiredActionFactory
```

This services/ file is used by Keycloak to scan the providers it has to load into the system.

To deploy this jar, just copy it to the providers directory.

**Implement the RequiredActionProvider**

Required actions must first implement the RequiredActionProvider interface. The RequiredActionProvider.requiredActionChallenge() is the initial call by the flow manager into the required action. This method is responsible for rendering the HTML form that will drive the required action.

```java
    @Override
    public void requiredActionChallenge(RequiredActionContext context) {
        Response challenge = context.form().createForm("secret_question_config.ftl");
        context.challenge(challenge);

    }
```

You see that RequiredActionContext has similar methods to AuthenticationFlowContext. The form() method allows you to render the page from a Freemarker template. The action URL is preset by the call to this form() method. You just need to reference it within your HTML form. I’ll show you this later.

The challenge() method notifies the flow manager that a required action must be executed.

The next method is responsible for processing input from the HTML form of the required action. The action URL of the form will be routed to the RequiredActionProvider.processAction() method

```java
    @Override
    public void processAction(RequiredActionContext context) {
        String answer = (context.getHttpRequest().getDecodedFormParameters().getFirst("answer"));
        UserCredentialValueModel model = new UserCredentialValueModel();
        model.setValue(answer);
        model.setType(SecretQuestionAuthenticator.CREDENTIAL_TYPE);
        context.getUser().updateCredentialDirectly(model);
        context.success();
    }
```

The answer is pulled out of the form post. A UserCredentialValueModel is created and the type and value of the credential are set. Then UserModel.updateCredentialDirectly() is invoked. Finally, RequiredActionContext.success() notifies the container that the required action was successful.

**Implement the RequiredActionFactory**

This class is really simple. It is just responsible for creating the required actin provider instance.

```java
public class SecretQuestionRequiredActionFactory implements RequiredActionFactory {

    private static final SecretQuestionRequiredAction SINGLETON = new SecretQuestionRequiredAction();

    @Override
    public RequiredActionProvider create(KeycloakSession session) {
        return SINGLETON;
    }


    @Override
    public String getId() {
        return SecretQuestionRequiredAction.PROVIDER_ID;
    }

    @Override
    public String getDisplayText() {
        return "Secret Question";
    }
```

The getDisplayText() method is just for the admin console when it wants to display a friendly name for the required action.

**Enable Required Action**

The final thing you have to do is go into the admin console. Click on the Authentication left menu. Click on the Required Actions tab. Click on the Register button and choose your new Required Action. Your new required action should now be displayed and enabled in the required actions list.

#### Modifying/Extending the Registration Form <a href="#modifying_extending_the_registration_form" id="modifying_extending_the_registration_form"></a>

It is entirely possible for you to implement your own flow with a set of Authenticators to totally change how regisration is done in Keycloak. But what you’ll usually want to do is just add a little bit of validation to the out of the box registration page. An additional SPI was created to be able to do this. It basically allows you to add validation of form elements on the page as well as to initialize UserModel attributes and data after the user has been registered. We’ll look at both the implementation of the user profile registration processing as well as the registration Google Recaptcha plugin.

**Implementation FormAction Interface**

The core interface you have to implement is the FormAction interface. A FormAction is responsible for rendering and processing a portion of the page. Rendering is done in the buildPage() method, validation is done in the validate() method, post validation operations are done in success(). Let’s first take a look at buildPage() method of the Recaptcha plugin.

```java
    @Override
    public void buildPage(FormContext context, LoginFormsProvider form) {
        AuthenticatorConfigModel captchaConfig = context.getAuthenticatorConfig();
        if (captchaConfig == null || captchaConfig.getConfig() == null
                || captchaConfig.getConfig().get(SITE_KEY) == null
                || captchaConfig.getConfig().get(SITE_SECRET) == null
                ) {
            form.addError(new FormMessage(null, Messages.RECAPTCHA_NOT_CONFIGURED));
            return;
        }
        String siteKey = captchaConfig.getConfig().get(SITE_KEY);
        form.setAttribute("recaptchaRequired", true);
        form.setAttribute("recaptchaSiteKey", siteKey);
        form.addScript("https://www.google.com/recaptcha/api.js");
    }
```

The Recaptcha buildPage() method is a callback by the form flow to help render the page. It receives a form parameter which is a LoginFormsProvider. You can add additional attributes to the form provider so that they can be displayed in the HTML page generated by the registration Freemarker template.

The code above is from the registration recaptcha plugin. Recaptcha requires some specific settings that must be obtained from configuration. FormActions are configured in the exact same was as Authenticators are. In this example, we pull the Google Recaptcha site key from configuration and add it as an attribute to the form provider. Our regstration template file can read this attribute now.

Recaptcha also has the requirement of loading a javascript script. You can do this by calling LoginFormsProvider.addScript() passing in the URL.

For user profile processing, there is no additional information that it needs to add to the form, so its buildPage() method is empty.

The next meaty part of this interface is the validate() method. This is called immediately upon receiving a form post. Let’s look at the Recaptcha’s plugin first.

```java
    @Override
    public void validate(ValidationContext context) {
        MultivaluedMap<String, String> formData = context.getHttpRequest().getDecodedFormParameters();
        List<FormMessage> errors = new ArrayList<>();
        boolean success = false;

        String captcha = formData.getFirst(G_RECAPTCHA_RESPONSE);
        if (!Validation.isBlank(captcha)) {
            AuthenticatorConfigModel captchaConfig = context.getAuthenticatorConfig();
            String secret = captchaConfig.getConfig().get(SITE_SECRET);

            success = validateRecaptcha(context, success, captcha, secret);
        }
        if (success) {
            context.success();
        } else {
            errors.add(new FormMessage(null, Messages.RECAPTCHA_FAILED));
            formData.remove(G_RECAPTCHA_RESPONSE);
            context.validationError(formData, errors);
            return;


        }
    }
```

Here we obtain the form data that the Recaptcha widget adds to the form. We obtain the Recaptcha secret key from configuration. We then validate the recaptcha. If successful, ValidationContext.success() is called. If not, we invoke ValidationContext.validationError() passing in the formData (so the user doesn’t have to re-enter data), we also specify an error message we want displayed. The error message must point to a message bundle property in the internationalized message bundles. For other registration extensions validate() might be validating the format of a form element, i.e. an alternative email attribute.

Let’s also look at the user profile plugin that is used to validate email address and other user information when registering.

```java
    @Override
    public void validate(ValidationContext context) {
        MultivaluedMap<String, String> formData = context.getHttpRequest().getDecodedFormParameters();
        List<FormMessage> errors = new ArrayList<>();

        String eventError = Errors.INVALID_REGISTRATION;

        if (Validation.isBlank(formData.getFirst((RegistrationPage.FIELD_FIRST_NAME)))) {
            errors.add(new FormMessage(RegistrationPage.FIELD_FIRST_NAME, Messages.MISSING_FIRST_NAME));
        }

        if (Validation.isBlank(formData.getFirst((RegistrationPage.FIELD_LAST_NAME)))) {
            errors.add(new FormMessage(RegistrationPage.FIELD_LAST_NAME, Messages.MISSING_LAST_NAME));
        }

        String email = formData.getFirst(Validation.FIELD_EMAIL);
        if (Validation.isBlank(email)) {
            errors.add(new FormMessage(RegistrationPage.FIELD_EMAIL, Messages.MISSING_EMAIL));
        } else if (!Validation.isEmailValid(email)) {
            formData.remove(Validation.FIELD_EMAIL);
            errors.add(new FormMessage(RegistrationPage.FIELD_EMAIL, Messages.INVALID_EMAIL));
        }

        if (context.getSession().users().getUserByEmail(email, context.getRealm()) != null) {
            formData.remove(Validation.FIELD_EMAIL);
            errors.add(new FormMessage(RegistrationPage.FIELD_EMAIL, Messages.EMAIL_EXISTS));
        }

        if (errors.size() > 0) {
            context.validationError(formData, errors);
            return;

        } else {
            context.success();
        }
    }
```

As you can see, this validate() method of user profile processing makes sure that the email, first, and last name are filled in in the form. It also makes sure that email is in the right format. If any of these validations fail, an error message is queued up for rendering. Any fields in error are removed from the form data. Error messages are represented by the FormMessage class. The first parameter of the constructor of this class takes the HTML element id. The input in error will be highlighted when the form is re-rendered. The second parameter is a message reference id. This id must correspond to a property in one of the localized message bundle files. in the theme.

After all validations have been processed then, the form flow then invokes the FormAction.success() method. For recaptcha this is a no-op, so we won’t go over it. For user profile processing, this method fills in values in the registered user.

```java
    @Override
    public void success(FormContext context) {
        UserModel user = context.getUser();
        MultivaluedMap<String, String> formData = context.getHttpRequest().getDecodedFormParameters();
        user.setFirstName(formData.getFirst(RegistrationPage.FIELD_FIRST_NAME));
        user.setLastName(formData.getFirst(RegistrationPage.FIELD_LAST_NAME));
        user.setEmail(formData.getFirst(RegistrationPage.FIELD_EMAIL));
    }
```

Pretty simple implementation. The UserModel of the newly registered user is obtained from the FormContext. The appropriate methods are called to initialize UserModel data.

Finally, you are also required to define a FormActionFactory class. This class is implemented similarly to AuthenticatorFactory, so we won’t go over it.

**Packaging the Action**

You will package your classes within a single jar. This jar must contain a file named `org.keycloak.authentication.FormActionFactory` and must be contained in the `META-INF/services/` directory of your jar. This file must list the fully qualified classname of each FormActionFactory implementation you have in the jar. For example:

```java
org.keycloak.authentication.forms.RegistrationProfile
org.keycloak.authentication.forms.RegistrationRecaptcha
```

This services/ file is used by Keycloak to scan the providers it has to load into the system.

To deploy this jar, just copy it to the providers directory.

**Adding FormAction to the Registration Flow**

Adding an FormAction to a registration page flow must be done in the admin console. If you go to the Authentication menu item and go to the Flow tab, you will be able to view the currently defined flows. You cannot modify an built in flows, so, to add the Authenticator we’ve created you have to copy an existing flow or create your own. I’m hoping the UI is intuitive enough so that you can figure out for yourself how to create a flow and add the FormAction.

Basically you’ll have to copy the registration flow. Then click Actions menu to the right of the Registration Form, and pick "Add Execution" to add a new execution. You’ll pick the FormAction from the selection list. Make sure your FormAction comes after "Registration User Creation" by using the down errors to move it if your FormAction isn’t already listed after "Registration User Creation". You want your FormAction to come after user creation because the success() method of Regsitration User Creation is responsible for creating the new UserModel.

After you’ve created your flow, you have to bind it to registration. If you go to the Authentication menu and go to the Bindings tab you will see options to bind a flow to the browser, registration, or direct grant flow.

#### Modifying Forgot Password/Credential Flow <a href="#modifying_forgot_password_credential_flow" id="modifying_forgot_password_credential_flow"></a>

Keycloak also has a specific authentication flow for forgot password, or rather credential reset initiated by a user. If you go to the admin console flows page, there is a "reset credentials" flow. By default, Keycloak asks for the email or username of the user and sends an email to them. If the user clicks on the link, then they are able to reset both their password and OTP (if an OTP has been set up). You can disable automatic OTP reset by disabling the "Reset OTP" authenticator in the flow.

You can add additional functionality to this flow as well. For example, many deployments would like for the user to answer one or more secret questions in additional to sending an email with a link. You could expand on the secret question example that comes with the distro and incorporate it into the reset credential flow.

One thing to note if you are extending the reset credentials flow. The first "authenticator" is just a page to obtain the username or email. If the username or email exists, then the AuthenticationFlowContext.getUser() will return the located user. Otherwise this will be null. This form **WILL NOT** re-ask the user to enter in an email or username if the previous email or username did not exist. You need to prevent attackers from being able to guess valid users. So, if AuthenticationFlowContext.getUser() returns null, you should proceed with the flow to make it look like a valid user was selected. I suggest that if you want to add secret questions to this flow, you should ask these questions after the email is sent. In other words, add your custom authenticator after the "Send Reset Email" authenticator.

#### Modifying First Broker Login Flow <a href="#modifying_first_broker_login_flow" id="modifying_first_broker_login_flow"></a>

First Broker Login flow is used during first login with some identity provider. Term `First Login` means that there is not yet existing Keycloak account linked with the particular authenticated identity provider account. For more details about this flow see the `Identity Brokering` chapter in [Server Administration](https://keycloak.gitbooks.io/documentation/content/server\_admin/index.html) .

#### Authentication of clients <a href="#client_authentication" id="client_authentication"></a>

Keycloak actually supports pluggable authentication for [OpenID Connect](http://openid.net/specs/openid-connect-core-1\_0.html) client applications. Authentication of client (application) is used under the hood by the Keycloak adapter during sending any backchannel requests to the Keycloak server (like the request for exchange code to access token after successful authentication or request to refresh token). But the client authentication can be also used directly by you during `Direct Access grants` (represented by OAuth2 `Resource Owner Password Credentials Flow`) or during `Service account` authentication (represented by OAuth2 `Client Credentials Flow`).

For more details about Keycloak adapter and OAuth2 flows see [Securing Applications and Services Guide](https://keycloak.gitbooks.io/documentation/content/securing\_apps/index.html).

**Default implementations**

Actually Keycloak has 2 builtin implementations of client authentication:

Traditional authentication with client\_id and client\_secret

This is default mechanism mentioned in the [OpenID Connect](http://openid.net/specs/openid-connect-core-1\_0.html) or [OAuth2](http://tools.ietf.org/html/rfc6749) specification and Keycloak supports it since it’s early days. The public client needs to include `client_id` parameter with it’s ID in the POST request (so it’s defacto not authenticated) and the confidential client needs to include `Authorization: Basic` header with the clientId and clientSecret used as username and password.

Authentication with signed JWT

This is based on the [JWT Bearer Token Profiles for OAuth 2.0](https://tools.ietf.org/html/rfc7523) specification. The client/adapter generates the [JWT](https://tools.ietf.org/html/rfc7519) and signs it with his private key. The Keycloak then verifies the signed JWT with the client’s public key and authenticates client based on it.

See the demo example and especially the `examples/preconfigured-demo/product-app` for the example application showing the application using client authentication with signed JWT.

**Implement your own client authenticator**

For plug your own client authenticator, you need to implement few interfaces on both client (adapter) and server side.

Client side

Here you need to implement `org.keycloak.adapters.authentication.ClientCredentialsProvider` and put the implementation either to:

* your WAR file into WEB-INF/classes . But in this case, the implementation can be used just for this single WAR application
* Some JAR file, which will be added into WEB-INF/lib of your WAR
* Some JAR file, which will be used as jboss module and configured in jboss-deployment-structure.xml of your WAR. In all cases, you also need to create the file `META-INF/services/org.keycloak.adapters.authentication.ClientCredentialsProvider` either in the WAR or in your JAR.

Server side

Here you need to implement `org.keycloak.authentication.ClientAuthenticatorFactory` and `org.keycloak.authentication.ClientAuthenticator` . You also need to add the file `META-INF/services/org.keycloak.authentication.ClientAuthenticatorFactory` with the name of the implementation classes. See [authenticators](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_development/topics/auth-spi.html#\_auth\_spi\_walkthrough) for more details.

\
