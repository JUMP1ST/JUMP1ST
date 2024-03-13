# IDP Keys sub element

The Keys sub element of IDP is only used to define the certificate or public key to use to verify documents signed by the IDP. It is defined in the same way as the [SP’s Keys element](https://wjw465150.gitbooks.io/keycloak-documentation/content/securing\_apps/topics/saml/java/general-config/sp-keys.html#\_saml-sp-keys). But again, you only have to define one certificate or public key reference. Note that, if both IDP and SP are realized by Keycloak server and adapter, respectively, there is no need to specify the keys for signature validation, see below.

It is possible to configure SP to obtain public keys for IDP signature validation from published certificates automatically, provided both SP and IDP are implemented by Keycloak. This is done by removing all declarations of signature validation keys in Keys sub element. If the Keys sub element would then remain empty, it can be omitted completely. The keys are then automatically obtained by SP from SAML descriptor, location of which is derived from SAML endpoint URL specified in the [IDP SingleSignOnService sub element](https://wjw465150.gitbooks.io/keycloak-documentation/content/securing\_apps/topics/saml/java/general-config/idp\_singlesignonservice\_subelement.html#\_sp-idp-singlesignonservice). Settings of the HTTP client that is used for SAML descriptor retrieval usually needs no additional configuration, however it can be configured in the [IDP HttpClient sub element](https://wjw465150.gitbooks.io/keycloak-documentation/content/securing\_apps/topics/saml/java/general-config/idp\_httpclient\_subelement.html#\_sp-idp-httpclient).

It is also possible to specify multiple keys for signature verification. This is done by declaring multiple Key elements within Keys sub element that have `signing` attribute set to `true`. This is useful for example in situation when the IDP signing keys are rotated: There is usually a transition period when new SAML protocol messages and assertions are signed with the new key but those signed by previous key should still be accepted.

It is not possible to configure Keycloak to both obtain the keys for signature verification automatically and define additional static signature verification keys.

```xml
       <IDP entityID="idp">
            ...
            <Keys>
                <Key signing="true">
                    <KeyStore resource="/WEB-INF/keystore.jks" password="store123">
                        <Certificate alias="demo"/>
                    </KeyStore>
                </Key>
            </Keys>
        </IDP>
```
