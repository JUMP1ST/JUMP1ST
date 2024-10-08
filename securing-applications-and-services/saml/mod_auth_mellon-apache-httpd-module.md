# mod\_auth\_mellon Apache HTTPD Module

The [mod\_auth\_mellon](https://github.com/UNINETT/mod\_auth\_mellon) module is an Apache HTTPD plugin for SAML. If your language/environment supports using Apache HTTPD as a proxy, then you can use mod\_auth\_mellon to secure your web application with SAML. For more details on this module see the _mod\_auth\_mellon_ Github repo.

To configure mod\_auth\_mellon you’ll need:

* An Identity Provider (IdP) entity descriptor XML file, which describes the connection to Keycloak or another SAML IdP
* An SP entity descriptor XML file, which describes the SAML connections and configuration for the application you are securing.
* A private key PEM file, which is a text file in the PEM format that defines the private key the application uses to sign documents.
* A certificate PEM file, which is a text file that defines the certificate for your application.
* mod\_auth\_mellon-specific Apache HTTPD module configuration.

If you have already defined and registered the client application within a realm on the Keycloak application server, Keycloak can generate all the files you need except the Apache HTTPD module configuration.

To generate the Apache HTTPD module configuration, complete the following steps:

1.  Go to the Installation page of your SAML client and select the Mod Auth Mellon files option.

    mod\_auth\_mellon config download

    ![mod-auth-mellon-config-download.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/securing\_apps/book.images/mod-auth-mellon-config-download.png)
2. Click **Download** to download a zip file that contains the XML descriptor and PEM files you need.

**Configuring mod\_auth\_mellon with Keycloak**

There are two hosts involved:

* The host on which Keycloak is running, which will be referred to as $idp\_host because Keycloak is a SAML identity provider (IdP).
* The host on which the web application is running, which will be referred to as $sp\_host. In SAML an application using an IdP is called a service provider (SP).

All of the following steps need to performed on $sp\_host with root privileges.

**Installing the Packages**

To install the necessary packages, you will need:

* Apache Web Server (httpd)
* Mellon SAML SP add-on module for Apache
* Tools to create X509 certificates

To install the necessary packages, run this command:

```
yum install httpd mod_auth_mellon mod_ssl openssl
```

**Creating a Configuration Directory for Apache SAML**

It is advisable to keep configuration files related to Apache’s use of SAML in one location.

Create a new directory named saml2 located under the Apache configuration root /etc/httpd:

```
mkdir /etc/httpd/saml2
```

**Configuring the Mellon Service Provider**

Configuration files for Apache add-on modules are located in the /etc/httpd/conf.d directory and have a file name extension of .conf. You need to create the /etc/httpd/conf.d/mellon.conf file and place Mellon’s configuration directives in it.

Mellon’s configuration directives can roughly be broken down into two classes of information:

* Which URLs to protect with SAML authentication
* What SAML parameters will be used when a protected URL is referenced.

Apache configuration directives typically follow a hierarchical tree structure in the URL space, which are known as locations. You need to specify one or more URL locations for Mellon to protect. You have flexibility in how you add the configuration parameters that apply to each location. You can either add all the necessary parameters to the location block or you can add Mellon parameters to a common location high up in the URL location hierarchy that specific protected locations inherit (or some combination of the two). Since it is common for an SP to operate in the same way no matter which location triggers SAML actions, the example configuration used here places common Mellon configuration directives in the root of the hierarchy and then specific locations to be protected by Mellon can be defined with minimal directives. This strategy avoids duplicating the same parameters for each protected location.

This example has just one protected location: https://$sp\_host/protected.

To configure the Mellon service provider, complete the following steps:

1. Create the file /etc/httpd/conf.d/mellon.conf with this content:

```xml
 <Location / >
    MellonEnable info
    MellonEndpointPath /mellon/
    MellonSPMetadataFile /etc/httpd/saml2/mellon_metadata.xml
    MellonSPPrivateKeyFile /etc/httpd/saml2/mellon.key
    MellonSPCertFile /etc/httpd/saml2/mellon.crt
    MellonIdPMetadataFile /etc/httpd/saml2/idp_metadata.xml
 </Location>
 <Location /private >
    AuthType Mellon
    MellonEnable auth
    Require valid-user
 </Location>
```

| Note | Some of the files referenced in the code above are created in later steps. |
| ---- | -------------------------------------------------------------------------- |

**Creating the Service Provider Metadata**

In SAML IdPs and SPs exchange SAML metadata, which is in XML format. The schema for the metadata is a standard, thus assuring participating SAML entities can consume each other’s metadata. You need:

* Metadata for the IdP that the SP utilizes
* Metadata describing the SP provided to the IdP

One of the components of SAML metadata is X509 certificates. These certificates are used for two purposes:

* Sign SAML messages so the receiving end can prove the message originated from the expected party.
* Encrypt the message during transport (seldom used because SAML messages typically occur on TLS-protected transports)

You can use your own certificates if you already have a Certificate Authority (CA) or you can generate a self-signed certificate. For simplicity in this example a self-signed certificate is used.

Because Mellon’s SP metadata must reflect the capabilities of the installed version of mod\_auth\_mellon, must be valid SP metadata XML, and must contain an X509 certificate (whose creation can be obtuse unless you are familiar with X509 certificate generation) the most expedient way to produce the SP metadata is to use a tool included in the mod\_auth\_mellon package (mellon\_create\_metadata.sh). The generated metadata can always be edited later because it is a text file. The tool also creates your X509 key and certificate.

SAML IdPs and SPs identify themselves using a unique name known as an EntityID. To use the Mellon metadata creation tool you need:

* The EntityID, which is typically the URL of the SP, and often the URL of the SP where the SP metadata can be retrieved
* The URL where SAML messages for the SP will be consumed, which Mellon calls the MellonEndPointPath.

To create the SP metadata, complete the following steps:

1.  Create a few helper shell variables:

    ```
    fqdn=`hostname`
    mellon_endpoint_url="https://${fqdn}/mellon"
    mellon_entity_id="${mellon_endpoint_url}/metadata"
    file_prefix="$(echo "$mellon_entity_id" | sed 's/[^A-Za-z.]/_/g' | sed 's/__*/_/g')"
    ```
2.  Invoke the Mellon metadata creation tool by running this command:

    ```
    /usr/libexec/mod_auth_mellon/mellon_create_metadata.sh $mellon_entity_id $mellon_endpoint_url
    ```
3.  Move the generated files to their destination (referenced in the /etc/httpd/conf.d/mellon.conf file created above):

    ```
    mv ${file_prefix}.cert /etc/httpd/saml2/mellon.crt
    mv ${file_prefix}.key /etc/httpd/saml2/mellon.key
    mv ${file_prefix}.xml /etc/httpd/saml2/mellon_metadata.xml
    ```

**Adding the Mellon Service Provider to the Keycloak Identity Provider**

Assumption: The Keycloak IdP has already been installed on the $idp\_host.

Keycloak supports multiple tenancy where all users, clients, and so on are grouped in what is called a realm. Each realm is independent of other realms. You can use an existing realm in your Keycloak, but this example shows how to create a new realm called test\_realm and use that realm.

All these operations are performed using the Keycloak administration web console. You must have the admin username and password for $idp\_host.

To complete the following steps:

1.  Open the Admin Console and log on by entering the admin username and password.

    After logging into the administration console there will be an existing realm. When Keycloak is first set up a root realm, master, is created by default. Any previously created realms are listed in the upper left corner of the administration console in a drop-down list.
2. From the realm drop-down list select **Add realm**.
3. In the Name field type `test_realm` and click **Create**.

**Adding the Mellon Service Provider as a Client of the Realm**

In Keycloak SAML SPs are known as clients. To add the SP we must be in the Clients section of the realm.

1. Click the Clients menu item on the left and click **Create** in the upper right corner to create a new client.

**Adding the Mellon SP Client**

To add the Mellon SP client, complete the following steps:

1. Set the client protocol to SAML. From the Client Protocol drop down list, select **saml**.
2. Provide the Mellon SP metadata file created above (/etc/httpd/saml2/mellon\_metadata.xml). Depending on where your browser is running you might have to copy the SP metadata from $sp\_host to the machine on which your browser is running so the browser can find the file.
3. Click **Save**.

**Editing the Mellon SP Client**

There are several client configuration parameters we suggest setting:

* Ensure "Force POST Binding" is On.
* Add paosResponse to the Valid Redirect URIs list:
  1. Copy the postResponse URL in "Valid Redirect URIs" and paste it into the empty add text fields just below the "+".
  2. Change "postResponse" to "paosResponse". (The paosResponse URL is needed for SAML ECP.)
  3. Click **Save** at the bottom.

Many SAML SPs determine authorization based on a user’s membership in a group. The Keycloak IdP can manage user group information but it does not supply the user’s groups unless the IdP is configured to supply it as a SAML attribute.

To configure the IdP to supply the user’s groups as as a SAML attribute, complete the following steps:

1. Click the Mappers tab of the client.
2. In the upper right corner of the Mappers page, click **Create**.
3. From the Mapper Type drop-down list select **Group list**.
4. Set Name to "group list."
5. Set the SAML attribute name to "groups."
6. Click **Save.**

The remaining steps are performed on $sp\_host.

**Retrieving the Identity Provider Metadata**

Now that you have created the realm on the IdP you need to retrieve the IdP metadata associated with it so the Mellon SP recognizes it. In the /etc/httpd/conf.d/mellon.conf file created previously, the MellonIdPMetadataFile is specified as /etc/httpd/saml2/idp\_metadata.xml but until now that file has not existed on $sp\_host. To get that file we will retrieve it from the IdP.

1.  Retrieve the file from the IdP by substituting $idp\_host with the correct value:

    ```
    curl -k -o /etc/httpd/saml2/idp_metadata.xml \
    https://$idp_host/auth/realms/test_realm/protocol/saml/descriptor
    ```

    Mellon is now fully configured.
2.  To run a syntax check for Apache configuration files:

    ```
    apachectl configtest
    ```

    | Note | Configtest is equivalent to the -t argument to apachectl. If the configuration test shows any errors, correct them before proceeding. |
    | ---- | ------------------------------------------------------------------------------------------------------------------------------------- |
3.  Restart the Apache server:

    ```
    systemctl restart httpd.service
    ```

You have now set up both Keycloak as a SAML IdP in the test\_realm and mod\_auth\_mellon as SAML SP protecting the URL $sp\_host/protected (and everything beneath it) by authenticating against the `$idp_host` IdP.
