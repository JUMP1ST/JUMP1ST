# Open redirectors

An attacker could use the end-user authorization endpoint and the redirect URI parameter to abuse the authorization server as an open redirector. An open redirector is an endpoint using a parameter to automatically redirect a user agent to the location specified by the parameter value without any validation. An attacker could utilize a user’s trust in an authorization server to launch a phishing attack.

Keycloak requires that all registered applications and clients register at least one redirection URI pattern. Any time a client asks Keycloak to perform a redirect (on login or logout for example), Keycloak will check the redirect URI vs. the list of valid registered URI patterns. It is important that clients and applications register as specific a URI pattern as possible to mitigate open redirector attacks.
