# Managing Resources

Resource servers can manage their resources remotely using a UMA-compliant endpoint.

```bash
http://${host}:${port}/auth/realms/${realm_name}/authz/protection/resource_set
```

This endpoint provides registration operations outlined as follows (entire path omitted for clarity):

* Create resource set description: POST /resource\_set
* Read resource set description: GET /resource\_set/{\_id}
* Update resource set description: PUT /resource\_set/{\_id}
* Delete resource set description: DELETE /resource\_set/{\_id}
* List resource set descriptions: GET /resource\_set
* List resource set descriptions using a filter: GET /resource\_set?filter=${filter}

For more information about the contract for each of these operations, see [UMA Resource Set Registration](https://docs.kantarainitiative.org/uma/rec-oauth-resource-reg-v1\_0\_1.html).
