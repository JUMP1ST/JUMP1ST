# Password guess: brute force attacks

A brute force attack happens when an attacker is trying to guess a user’s password. Keycloak has some limited brute force detection capabilities. If turned on, a user account will be temporarily disabled if a threshold of login failures is reached. To enable this feature go to the `Realm Settings` left menu item, click on the `Security Defenses` tab, then additional go to the `Brute Force Detection` sub-tab.

Brute Force Detection

![brute-force.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/brute-force.png)

The way this works is that if there are `Max Login Failures` during a period of `Failure Reset Time`, the account is temporarily disabled for the `Wait Increment` multiplied by the number of failures over the max. After `Failure Reset Time` is reached all failures are wiped clean. The `Max Wait` is the maximum amount of time an account can be disabled. Another preventive measure is that if there are subsequent login failures for one account that are too quick for a human to initiate the account will be disabled. This is controlled by the `Quick Login Check Milli Seconds` value. So, if there are two login failures for the same account within that value, the account will be disabled for `Minimum Quick Login Wait`.

The downside of Keycloak brute force detection is that the server becomes vulnerable to denial of service attacks. An attacker can simply try to guess passwords for any accounts it knows and these account will be disabled. Eventually we will expand this functionality to take client IP address into account when deciding whether to block a user.

A better option might be a tool like [Fail2Ban](http://www.fail2ban.org/). You can point this service at the Keycloak server’s log file. Keycloak logs every login failure and client IP address that had the failure. Fail2Ban can be used to modify firewalls after it detects an attack to block connections from specific IP addresses.

**Password Policies**

Another thing you should do to prevent password guess is to have a complex enough password policy to ensure that users pick hard to guess passwords. See the [Password Policies](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/authentication/password-policies.html#\_password-policies) chapter for more details.

The best way to prevent password guessing though is to set up the server to use a one-time-password (OTP).
