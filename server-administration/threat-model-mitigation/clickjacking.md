# Clickjacking

With clickjacking, a malicious site loads the target site in a transparent iFrame overlaid on top of a set of dummy buttons that are carefully constructed to be placed directly under important buttons on the target site. When a user clicks a visible button, they are actually clicking a button (such as a "login" button) on the hidden page. An attacker can steal a user’s authentication credentials and access their resources.

By default, every response by Keycloak sets some specific browser headers that can prevent this from happening. Specifically, it sets [X-FRAME\_OPTIONS](http://tools.ietf.org/html/rfc7034) and [Content-Security-Policy](http://www.w3.org/TR/CSP/). You should take a look at the definition of both of these headers as there is a lot of fine-grain browser access you can control. In the admin console you can specify the values these headers will have. Go to the `Realm Settings` left menu item and click the `Security Defenses` tab and make sure you are on the `Headers` sub-tab.

![security-headers.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/security-headers.png)

By default, Keycloak only sets up a _same-origin_ policy for iframes.
