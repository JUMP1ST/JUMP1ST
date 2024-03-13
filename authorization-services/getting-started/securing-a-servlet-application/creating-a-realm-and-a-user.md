# Creating a Realm and a User

The first step is to create a realm and a user in that realm. The realm consists of:

* A single user
* A single client application, which then becomes a [resource server](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/topics/overview/terminology.html#\_overview\_terminology) for which you need to enable authorization services.

To create a realm and a user complete the following steps:

1.  Create a realm with a name **hello-world-authz**. Once created, a page similar to the following is displayed:

    Realm hello-world-authz

    ![Realm hello-world-authz](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/getting-started/hello-world/create-realm.png)
2. Create a user for your newly created realm. Click **Users**. The user list page opens.
3. On the right side of the empty user list, click **Add User**.
4.  To create a new user, complete the **Username**, **Email**, **First Name**, and **Last Name** fields. Click the **User Enabled** switch to **On**, and then click **Save**.

    Add User

    ![Add User](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/getting-started/hello-world/create-user.png)
5.  Set a password for the user by clicking the **Credentials** tab.

    Set User Password

    ![Set User Password](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/getting-started/hello-world/reset-user-pwd.png)
6. Complete the **New Password** and **Password Confirmation** fields with a password and click the **Temporary** switch to **OFF**.
7. Click **Reset Password** to set the user’s password.

\
