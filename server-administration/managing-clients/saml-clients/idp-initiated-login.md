# IDP Initiated Login

IDP Initiated Login is a feature that allows you to set up an endpoint on the Keycloak server that will log you into a specific application/client. In the `Settings` tab for your client, you need to specify the `IDP Initiated SSO URL Name`. This is a simple string with no whitespace in it. After this you can reference your client at the following URL: `root/auth/realms/{realm}/protocol/saml/clients/{url-name}`

If your client requires a special relay state, you can also configure this on the `Settings` tab in the `IDP Initiated SSO Relay State` field. Alternatively, browsers can specify the relay state in a `RelayState` query parameter, i.e. `root/auth/realms/{realm}/protocol/saml/clients/{url-name}?RelayState=thestate`.

When using [identity brokering](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/clients/saml/idp-initiated-login.html#\_identity\_broker), it is possible to set up an IDP Initiated Login for a client from an external IDP. The actual client is set up for IDP Initiated Login at broker IDP as described above. The external IDP has to set up the client for application IDP Initiated Login that will point to a special URL pointing to the broker and representing IDP Initiated Login endpoint for a selected client at the brokering IDP. This means that in client settings at the external IDP:

* `IDP Initiated SSO URL Name` is set to a name that will be published as IDP Initiated Login initial point,
* `Assertion Consumer Service POST Binding URL` in the `Fine Grain SAML Endpoint Configuration` section has to be set to the following URL: `broker-root/auth/realms/{broker-realm}/broker/{idp-name}/endpoint/clients/{client-id}`, where:
  * _broker-root_ is base broker URL
  * _broker-realm_ is name of the realm at broker where external IDP is declared
  * _idp-name_ is name of the external IDP at broker
  * _client-id_ is the value of `IDP Initiated SSO URL Name` attribute of the SAML client defined at broker. It is this client, which will be made available for IDP Initiated Login from the external IDP.

Please note that you can import basic client settings from the brokering IDP into client settings of the external IDP - just use [SP Descriptor](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/topics/clients/saml/idp-initiated-login.html#\_identity\_broker\_saml\_sp\_descriptor) available from the settings of the identity provider in the brokering IDP, and add `clients/client-id` to the endpoint URL.
