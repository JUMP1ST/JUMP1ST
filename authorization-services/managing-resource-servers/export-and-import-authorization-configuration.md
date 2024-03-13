# Export and Import Authorization Configuration

The configuration settings for a resource server (or client) can be exported and downloaded. You can also import an existing configuration file for a resource server. Importing and exporting a configuration file is helpful when you want to create an initial configuration for a resource server or to update an existing configuration. The configuration file contains definitions for:

* Protected resources and scopes
* Policies
* Permissions

**Exporting a Configuration File**

To export a configuration file, complete the following steps:

1.  Navigate to the **Resource Server Settings** page.

    Resource Server Settings

    ![Resource Server Settings](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/resource-server/authz-settings.png)
2.  On this page, in the Export Settings section, click **Export**.

    Export Settings

    ![Export Settings](https://wjw465150.gitbooks.io/keycloak-documentation/content/authorization\_services/keycloak-images/resource-server/authz-export.png)

The configuration file is exported in JSON format and displayed in a text area, from which you can copy and paste. You can also click **Download** to download the configuration file and save it.

**Importing a Configuration File**

To import a configuration file for a resource server, click **Select file** to select a file containing the configuration you want to import.
