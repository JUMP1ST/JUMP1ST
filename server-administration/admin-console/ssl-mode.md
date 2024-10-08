# SSL Mode

Each realm has an SSL Mode associated with it. The SSL Mode defines the SSL/HTTPS requirements for interacting with the realm. Browsers and applications that interact with the realm must honor the SSL/HTTPS requirements defined by the SSL Mode or they will not be allowed to interact with the server.

| Warning | Keycloak is not set up by default to handle SSL/HTTPS. It is highly recommended that you either enable SSL on the Keycloak server itself or on a reverse proxy in front of the Keycloak server. |
| ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

To configure the SSL Mode of your realm, you need to click on the `Realm Settings` left menu item and go to the `Login` tab.

Login Tab

![](<../../.gitbook/assets/image (3).png>)\


The `Require SSL` option allows you to pick the SSL Mode you want. Here is an explanation of each mode:

external requests

Users can interact with Keycloak so long as they stick to private IP addresses like `localhost`, `127.0.0.1`, `10.0.x.x`, `192.168.x.x`, and `172..16.x.x`. If you try to access Keycloak from a non-private IP address you will get an error.

none

Keycloak does not require SSL. This should really only be used in development when you are playing around with things and don’t want to bother configuring SSL on your server.

all requests

Keycloak requires SSL for all IP addresses.
