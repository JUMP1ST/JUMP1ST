# Password Policies

Each new realm created has no password policies associated with it. Users can have as short, as long, as complex, as insecure a password, as they want. Simple settings are fine for development or learning Keycloak, but unacceptable in production environments. Keycloak has a rich set of password policies you can enable through the Admin Console.

Click on the `Authentication` left menu item and go to the `Password Policy` tab. Choose the policy you want to add in the right side drop down list box. This will add the policy in the table on the screen. Choose the parameters for the policy. Hit the `Save` button to store your changes.

Password Policy

![password-policy.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/password-policy.png)

After saving your policy, user registration and the Update Password required action will enforce your new policy. An example of a user failing the policy check:

Failed Password Policy

![failed-password-policy.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/failed-password-policy.png)

If the password policy is updated, an Update Password action must be set for every user. An automatic trigger is scheduled as a future enhancement.

**Password Policy Types**

Here’s an explanation of each policy type:

HashAlgorithm

Passwords are not stored as clear text. Instead they are hashed using standard hashing algorithms before they are stored or validated. The only built-in and default algorithm available is PBKDF2. See the [Server Development](https://keycloak.gitbooks.io/documentation/content/server\_development/index.html) on how to plug in your own algorithm. Note that if you do change the algorithm, password hashes will not change in storage until the next time the user logs in.

Hashing Iterations

This value specifies the number of times a password will be hashed before it is stored or verified. The default value is 20,000. This hashing is done in the rare case that a hacker gets access to your password database. Once they have access to the database, they can reverse engineer user passwords. The industry recommended value for this parameter changes every year as CPU power improves. A higher hashing iteration value takes more CPU power for hashing, and can impact performance. You’ll have to weigh what is more important to you. Performance or protecting your passwords stores. There may be more cost effective ways of protecting your password stores.

Digits

The number of digits required to be in the password string.

Lowercase Characters

The number of lower case letters required to be in the password string.

Uppercase Characters

The number of upper case letters required to be in the password string.

Special Characters

The number of special characters like '?!#%$' required to be in the password string.

Not Username

When set, the password is not allowed to be the same as the username.

Regular Expression

Define one or more Perl regular expression patterns that passwords must match.

Expire Password

The number of days for which the password is valid. After the number of days has expired, the user is required to change their password.

Not Recently Used

This policy saves a history of previous passwords. The number of old passwords stored is configurable. When a user changes their password they cannot use any stored passwords.

\
