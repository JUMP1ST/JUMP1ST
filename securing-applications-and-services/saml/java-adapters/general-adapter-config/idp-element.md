# IDP Element

Everything in the IDP element describes the settings for the identity provider (authentication server) the SP is communicating with.

```xml
<IDP entityID="idp"
     signaturesRequired="true"
     signatureAlgorithm="RSA_SHA1"
     signatureCanonicalizationMethod="http://www.w3.org/2001/10/xml-exc-c14n#">
...
</IDP>
```

Here are the attribute config options you can specify within the `IDP` element declaration.

entityID

This is the issuer ID of the IDP. This setting is _REQUIRED_.

signaturesRequired

If set to `true`, the client adapter will sign every document it sends to the IDP. Also, the client will expect that the IDP will be signing any documents sent to it. This switch sets the default for all request and response types, but you will see later that you have some fine grain control over this. This setting is _OPTIONAL_ and will default to `false`.

signatureAlgorithm

This is the signature algorithm that the IDP expects signed documents to use. Allowed values are: `RSA_SHA1`, `RSA_SHA256`, `RSA_SHA512`, and `DSA_SHA1`. This setting is _OPTIONAL_ and defaults to `RSA_SHA256`.

signatureCanonicalizationMethod

This is the signature canonicalization method that the IDP expects signed documents to use. This setting is _OPTIONAL_. The default value is `http://www.w3.org/2001/10/xml-exc-c14n#` and should be good for most IDPs.
