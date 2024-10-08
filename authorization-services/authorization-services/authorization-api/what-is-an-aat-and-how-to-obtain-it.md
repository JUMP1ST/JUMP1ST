# What is an AAT and How to Obtain It

An authorization API token (AAT) is a special OAuth2 access token with the scope **uma\_authorization**. When you create a user, Keycloak automatically assigns the role _uma\_authorization_ to the user. The _uma\_authorization_ role is a default realm role.

Default Role uma\_authorization

![Default Role uma\_authorization](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/service/rs-uma-authorization-role.png)

An AAT enables a client application to query the server for user permissions.

Client applications can obtain an AAT from Keycloak like any other OAuth2 access token. Usually, client applications obtain AATs after the user is successfully authenticated in Keycloak. By default, the _authorization\_code_ grant type is used to authenticate users, and the server will issue an OAuth2 access token to the client application acting on their behalf.

The example below uses the Resource Owner Password Credentials Grant Type to request an AAT:

```bash
curl -X POST \
    -H "Authorization: Basic aGVsbG8td29ybGQtYXV0aHotc2VydmljZTpwYXNzd29yZA==" \
    -H "Content-Type: application/x-www-form-urlencoded" \
    -d 'username=${username}&password=${user_password}&grant_type=password' \
    "http://localhost:8080/auth/realms/${realm_name}/protocol/openid-connect/token"
```

As a result, the server response is:

```json
{
  "access_token": ${AAT},
  "expires_in": 300,
  "refresh_expires_in": 1800,
  "refresh_token": ${refresh_token},
  "token_type": "bearer",
  "id_token": ${id_token},
  "not-before-policy": 0,
  "session_state": "3cad2afc-855b-47b7-8e4d-a21c66e312fb"
}
```
