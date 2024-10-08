# Setting Up TLS/HTTPS

When the server is using HTTPS, ensure your adapter is configured as follows:

keycloak.json

```json
{
  "truststore": "path_to_your_trust_store",
  "truststore-password": "trust_store_password"
}
```

The configuration above enables TLS/HTTPS to the Authorization Client, making possible to access a Keycloak Server remotely using the HTTPS scheme.

| Note | It is strongly recommended that you enable TLS/HTTPS when accessing the Keycloak Server endpoints. |
| ---- | -------------------------------------------------------------------------------------------------- |

\
