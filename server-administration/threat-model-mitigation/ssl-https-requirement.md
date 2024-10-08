# SSL/HTTPS Requirement

If you do not use SSL/HTTPS for all communication between the Keycloak auth server and the clients it secures, you will be very vulnerable to man in the middle attacks. OAuth 2.0/OpenID Connect uses access tokens for security. Without SSL/HTTPS, attackers can sniff your network and obtain an access token. Once they have an access token they can do any operation that the token has been given permission for.

Keycloak has [three modes for SSL/HTTPS](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/realms/ssl.html#\_ssl\_modes). SSL can be hard to set up, so out of the box, Keycloak allows non-HTTPS communication over private IP addresses like localhost, 192.168.x.x, and other private IP addresses. In production, you should make sure SSL is enabled and required across the board.

On the adapter/client side, Keycloak allows you to turn off the SSL trust manager. The trust manager ensures identity the client is talking to. It checks the DNS domain name against the server’s certificate. In production you should make sure that each of your client adapters is configured to use a truststore. Otherwise you are vulnerable to DNS man in the middle attacks.
