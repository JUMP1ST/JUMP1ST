# Troubleshooting

The best way to troubleshoot problems is to turn on debugging for SAML in both the client adapter and Keycloak Server. Using your logging framework, set the log level to `DEBUG` for the `org.keycloak.saml` package. Turning this on allows you to see the SAML requests and response documents being sent to and from the server.
