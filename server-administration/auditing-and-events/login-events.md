# Login Events

Login events occur for things like when a user logs in successfully, when somebody enters in a bad password, or when a user account is updated. Every single event that happens to a user can be recorded and viewed. By default, no events are stored or viewed in the Admin Console. Only error events are logged to the console and the server’s log file. To start persisting you’ll need to enable storage. Go to the `Events` left menu item and select the `Config` tab.

Event Configuration

![login-events-config.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/login-events-config.png)

To start storing events you’ll need to turn the `Save Events` switch to on under the `Login Events Settings`.

Save Events

![login-events-settings.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/login-events-settings.png)

The `Saved Types` field allows you to specify which event types you want to store in the event store. The `Clear events` button allows you to delete all the events in the database. The `Expiration` field allows you to specify how long you want to keep events stored. Once you’ve enabled storage of login events and decided on your settings, don’t forget to click the `Save` button on the bottom of this page.

To view events, go to the `Login Events` tab.

Login Events

![login-events.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/login-events.png)

As you can see, there’s a lot of information stored and, if you are storing every event, there are a lot of events stored for each login action. The `Filter` button on this page allows you to filter which events you are actually interested in.

Login Event Filter

![login-events-filter.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/login-events-filter.png)

In this screenshot, we’re filtering only `Login` events. Clicking the `Update` button runs the filter.

**Event Types**

Login events:

* Login - A user has logged in.
* Register - A user has registered.
* Logout - A user has logged out.
* Code to Token - An application/client has exchanged a code for a token.
* Refresh Token - An application/client has refreshed a token.

Account events:

* Social Link - An account has been linked to a social provider.
* Remove Social Link - A social provider has been removed from an account.
* Update Email - The email address for an account has changed.
* Update Profile - The profile for an account has changed.
* Send Password Reset - A password reset email has been sent.
* Update Password - The password for an account has changed.
* Update TOTP - The TOTP settings for an account have changed.
* Remove TOTP - TOTP has been removed from an account.
* Send Verify Email - An email verification email has been sent.
* Verify Email - The email address for an account has been verified.

For all events there is a corresponding error event.

**Event Listener**

Event listeners listen for events and perform an action based on that event. There are two built-in listeners that come with Keycloak: Logging Event Listener and Email Event Listener.

The Logging Event Listener writes to a log file whenever an error event occurs and is enabled by default. Here’s an example log message:

```
11:36:09,965 WARN  [org.keycloak.events] (default task-51) type=LOGIN_ERROR, realmId=master,
                    clientId=myapp,
                    userId=19aeb848-96fc-44f6-b0a3-59a17570d374, ipAddress=127.0.0.1,
                    error=invalid_user_credentials, auth_method=openid-connect, auth_type=code,
                    redirect_uri=http://localhost:8180/myapp,
                    code_id=b669da14-cdbb-41d0-b055-0810a0334607, username=admin
```

This logging is very useful if you want to use a tool like Fail2Ban to detect if there is a hacker bot somewhere that is trying to guess user passwords. You can parse the log file for `LOGIN_ERROR` and pull out the IP Address. Then feed this information into Fail2Ban so that it can help prevent attacks.

The Email Event Listener sends an email to the user’s account when an event occurs. The Email Event Listener only supports the following events at the moment:

* Login Error
* Update Password
* Update TOTP
* Remove TOTP

To enable the Email Listener go to the `Config` tab and click on the `Event Listeners` field. This will show a drop down list box where you can select email.

You can exclude one or more events by editing the `standalone.xml`, `standalone-ha.xml`, or `domain.xml` that comes with your distribution and adding for example:

```xml
<spi name="eventsListener">
  <provider name="email" enabled="true">
    <properties>
      <property name="exclude-events" value="[&quot;UPDATE_TOTP&quot;,&quot;REMOVE_TOTP&quot;]"/>
    </properties>
  </provider>
</spi>
```

See the [Server Installation and Configuration](https://keycloak.gitbooks.io/documentation/content/server\_installation/index.html) for more details on where the `standalone.xml`, `standalone-ha.xml`, or `domain.xml` file lives.
