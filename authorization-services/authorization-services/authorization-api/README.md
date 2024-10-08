# Authorization API

The Authorization API provides a UMA-compliant endpoint for obtaining authorization data from the server, where the authorization data represents the result of the evaluation of all permissions and authorization policies associated with the resources being requested.

Unlike the Protection API, any client application can access the Authorization API endpoint, which requires a special OAuth2 access token called an authorization API token (AAT). In UMA, an AAT is a token with the scope **uma\_authorization**.
