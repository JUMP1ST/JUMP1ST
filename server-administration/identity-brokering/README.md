# Identity Brokering

An Identity Broker is an intermediary service that connects multiple service providers with different identity providers. As an intermediary service, the identity broker is responsible for creating a trust relationship with an external identity provider in order to use its identities to access internal services exposed by service providers.

From a user perspective, an identity broker provides a user-centric and centralized way to manage identities across different security domains or realms. An existing account can be linked with one or more identities from different identity providers or even created based on the identity information obtained from them.

An identity provider is usually based on a specific protocol that is used to authenticate and communicate authentication and authorization information to their users. It can be a social provider such as Facebook, Google or Twitter. It can be a business partner whose users need to access your services. Or it an be a cloud-based identity service that you want to integrate with.

Usually, identity providers are based on the following protocols:

* `SAML v2.0`
* `OpenID Connect v1.0`
* `OAuth v2.0`

In the next sections we’ll see how to configure and use Keycloak as an identity broker, covering some important aspects such as:

* `Social Authentication`
* `OpenID Connect v1.0 Brokering`
* `SAML v2.0 Brokering`
* `Identity Federation`
