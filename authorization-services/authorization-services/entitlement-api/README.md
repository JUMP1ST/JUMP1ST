# Entitlement API

The Entitlement API provides a 1-legged protocol for obtaining authorization data from the server, where the authorization data represents the result of the evaluation of all permissions and authorization policies associated with the resources being requested.

Unlike the _Authorization API_, the Entitlement API is not UMA-compliant and does not require permission tickets.

The purpose of this API is provide a more lightweight API for obtaining authorization data, where a client in possession of a valid OAuth2 access token is able to obtain the necessary authorization data on behalf of its users.
