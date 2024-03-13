# Distribution Directory Structure

This chapter walks you through the directory structure of the server distribution.

distribution directory structure

![distribution](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/keycloak-images/files.png)

Let’s examine the purpose of some of the directories:

_bin/_

This contains various scripts to either boot the server or perform some other management action on the server.

_domain/_

This contains configuration files and working directory when running Keycloak in [domain mode](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/operating-mode/domain.html#\_domain-mode).

_modules/_

These are all the Java libraries used by the server.

_providers/_

If you are writing extensions to keycloak, you can put your extensions here. See the [Server Development](https://keycloak.gitbooks.io/documentation/content/server\_development/index.html) for more information on this.

_standalone/_

This contains configuration files and working directory when running Keycloak in [standalone mode](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/operating-mode/standalone.html#\_standalone-mode).

_themes/_

This directory contains all the html, style sheets, javascript files, and images used to display any UI screen displayed by the server. Here you can modify an existing theme or create your own. See the [Server Development](https://keycloak.gitbooks.io/documentation/content/server\_development/index.html) for more information on this.
