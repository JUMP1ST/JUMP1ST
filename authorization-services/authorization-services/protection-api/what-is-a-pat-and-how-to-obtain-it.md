# What is a PAT and How to Obtain It

A **protection API token** (PAT) is a special OAuth2 access token with a scope defined as **uma\_protection**. When you create a resource server, Keycloak automatically creates a role, _uma\_protection_, for the corresponding client application and associates it with the client’s service account.

Service Account granted with **uma\_protection** role

![Service Account granted with uma\_protection role](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/service/rs-uma-protection-role.png)

Resource servers can obtain a PAT from Keycloak like any other OAuth2 access token. For example, using curl:

```bash
curl -X POST \
    -H "Authorization: Basic aGVsbG8td29ybGQtYXV0aHotc2VydmljZTpwYXNzd29yZA==" \
    -H "Content-Type: application/x-www-form-urlencoded" \
    -d 'grant_type=client_credentials' \
    "http://localhost:8080/auth/realms/${realm_name}/protocol/openid-connect/token"
```

The example above is using the **client\_credentials** grant type to obtain a PAT from the server. As a result, the server returns a response similar to the following:

```bash
{
  "access_token": ${PAT},
  "expires_in": 300,
  "refresh_expires_in": 1800,
  "refresh_token": ${refresh_token},
  "token_type": "bearer",
  "id_token": ${id_token},
  "not-before-policy": 0,
  "session_state": "ccea4a55-9aec-4024-b11c-44f6f168439e"
}
```

| Note | Keycloak can authenticate your client application in different ways. For simplicity, the **client\_credentials** grant type is used here, which requires a _client\_id_ and a _client\_secret_. You can choose to use any supported authentication method. |
| ---- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
