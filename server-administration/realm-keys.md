# Realm Keys

The authentication protocols that are used by Keycloak require cryptographic signatures and sometimes encryption. Keycloak uses asymmetric key pairs, a private and public key, to accomplish this.

Keycloak has a single active keypair at a time, but can have several passive keys as well. The active keypair is used to create new signatures, while the passive keypairs can be used to verify previous signatures. This makes it possible to regularly rotate the keys without any downtime or interruption to users.

When a realm is created a key pair and a self-signed certificate is automatically generated.

To view the active keys for a realm select the realm in the admin console click on `Realm settings` then `Keys`. This will show the currently active keys for the realm. Keycloak currently only supports RSA signatures so there is only one active keypair. In the future as more signature algorithms are added there will be more active keypairs.

To view all available keys select `All`. This will show all active, passive and disabled keys. A keypair can have the status `Active`, but still not be selected as the currently active keypair for the realm. The selected active pair which is used for signatures is selected based on the first key provider sorted by priority that is able to provide an active keypair.

**Rotating keys**

It’s recommended to regularly rotate keys. To do so you should start by creating new keys with a higher priority than the existing active keys. Or create new keys with the same priority and making the previous keys passive.

Once new keys are available all new tokens and cookies will be signed with the new keys. When a user authenticates to an application the SSO cookie is updated with the new signature. When OpenID Connect tokens are refreshed new tokens are signed with the new keys. This means that over time all cookies and tokens will use the new keys and after a while the old keys can be removed.

How long you wait to delete old keys is a tradeoff between security and making sure all cookies and tokens are updated. In general it should be acceptable to drop old keys after a few weeks. Users that have not been active in the period between the new keys where added and the old keys removed will have to re-authenticate.

This also applies to offline tokens. To make sure they are updated the applications need to refresh the tokens before the old keys are removed.

As a guideline it may be a good idea to create new keys every 3-6 months and delete old keys 1-2 months after the new keys where created.

**Adding a generated keypair**

To add a new generated keypair select `Providers` and choose `rsa-generated` from the dropdown. You can change the priority to make sure the new keypair becomes the active keypair. You can also change the `keysize` if you want smaller or larger keys (default is 2048, supported values are 1024, 2048 and 4096).

Click `Save` to add the new keys. This will generated a new keypair including a self-signed certificate.

Changing the priority for a provider will not cause the keys to be re-generated, but if you want to change the keysize you can edit the provider and new keys will be generated.

**Adding an existing keypair and certificate**

To add a keypair and certificate obtained elsewhere select `Providers` and choose `rsa` from the dropdown. You can change the priority to make sure the new keypair becomes the active keypair.

Click on `Select file` for `Private RSA Key` to upload your private key. The file should be encoded in PEM format. You don’t need to upload the public key as it is automatically extracted from the private key.

If you have a signed certificate for the keys click on `Select file` next to `X509 Certificate`. If you don’t have one you can skip this and a self-signed certificate will be generated.

**Loading keys from a Java Keystore**

To add a keypair and certificate stored in a Java Keystore file on the host select `Providers` and choose `java-keystore` from the dropdown. You can change the priority to make sure the new keypair becomes the active keypair.

Fill in the values for `Keystore`, `Keystore Password`, `Key Alias` and `Key Password` and click on `Save`.

**Making keys passive**

Locate the keypair in `Active` or `All` then click on the provider in the `Provider` column. This will take you to the configuration screen for the key provider for the keys. Click on `Active` to turn it `OFF`, then click on `Save`. The keys will no longer be active and can only be used for verifying signatures.

**Disabling keys**

Locate the keypair in `Active` or `All` then click on the provider in the `Provider` column. This will take you to the configuration screen for the key provider for the keys. Click on `Enabled` to turn it `OFF`, then click on `Save`. The keys will no longer be enabled.

Alternatively, you can delete the provider from the `Providers` table.

**Compromised keys**

Keycloak has the signing keys stored just locally and they are never shared with the client applications, users or other entities. However if you think that your realm signing key was compromised, you should first generate new keypair as described above and then immediatelly remove the compromised keypair.

Then to ensure that client applications won’t accept the tokens signed by the compromised key, you should update and push not-before policy for the realm, which is doable from the admin console. Pushing new policy will ensure that client applications won’t accept the existing tokens signed by the compromised key, but also the client application will be forced to download new keypair from the Keycloak, hence the tokens signed by the compromised key won’t be valid anymore. Note that your REST and confidential clients must have set `Admin URL`, so that Keycloak is able to send them the request about pushed not-before policy.
