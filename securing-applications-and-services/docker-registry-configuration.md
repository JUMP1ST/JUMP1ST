# Docker Registry Configuration

| Note | Docker authentication is disabled by default. To enable see [Profiles](https://keycloak.gitbooks.io/documentation/content/server\_installation/topics/profiles.html). |
| ---- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

This section describes how you can configure a Docker registry to use Keycloak as its authentication server.

Fore more information on how to set up and configure a Docker registry, see the [Docker Registry Configuration Guide](https://docs.docker.com/registry/configuration/).

#### Docker Registry Configuration File Installation <a href="#docker_registry_configuration_file_installation" id="docker_registry_configuration_file_installation"></a>

For users with more advanced docker registry configurations, it is generally recommended to provide your own registry configuration file. The Keycloak docker provider supports this mechanism via the _Registry Config File_ Format Option. Choosing this option will generate output similar to the following:

```
auth:
  token:
    realm: http://localhost:8080/auth/auth/realms/master/protocol/docker-v2/auth
    service: docker-test
    issuer: http://localhost:8080/auth/auth/realms/master
```

This output can then be copied into any existing registry config file. See the [registry config file specification](https://docs.docker.com/registry/configuration/) for more information on how the file should be set up, or start with href:https://github.com/docker/distribution/blob/master/cmd/registry/config-example.yml\[a basic example].

| Warning | Don’t forget to configure the `rootcertbundle` field with the location of the Keycloak realm’s pulic certificate. The auth configuration will not work without this argument. |
| ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

#### Docker Registry Environment Variable Override Installation <a href="#docker_registry_environment_variable_override_installation" id="docker_registry_environment_variable_override_installation"></a>

Often times it is appropriate to use a simple environment variable override for develop or POC Docker registries. While this apporach is usually not recommended for production use, it can be helpful when one requires quick-and-dirty way to stand up a registry. Simply use the _Variable Override_ Format Option from the client installation tab, and an output should appear like the one below:

```
REGISTRY_AUTH_TOKEN_REALM: http://localhost:8080/auth/auth/realms/master/protocol/docker-v2/auth
REGISTRY_AUTH_TOKEN_SERVICE: docker-test
REGISTRY_AUTH_TOKEN_ISSUER: http://localhost:8080/auth/auth/realms/master
```

| Warning | Don’t forget to configure the `REGISTRY_AUTH_TOKEN_ROOTCERTBUNDLE` override with the location of the Keycloak realm’s pulic certificate. The auth configuration will not work without this argument. |
| ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

#### Docker Compose YAML File <a href="#docker_compose_yaml_file" id="docker_compose_yaml_file"></a>

| Warning | This installation method is meant to be an easy way to get a docker registry authenticating against a keycloak server. It is intended for development purposes only and should never be used in a production or production-like environment. |
| ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

The zip file installation mechanism provides a quickstart for developers who want to understand how the keycloak server can interact with the docker registry. In order to configure:

1. From the desired realm, create a client configuration. At this point you won’t have a docker registry - the quickstart will take care of that part.
2. Choose the "Docker Compose YAML" option from the installation tab and download the .zip file
3. Unzip the archive to the desired location, and open the directory.
4. Start the docker registry with `docker-compose up`

INFO: it is recommended that you configure the docker registry client in a realm other than 'master', since the HTTP Basic auth flow will not present forms.

Once the above configuration has taken place, and the keycloak server and docker registry are running, docker authentication should be successful:

```
[user ~]# docker login localhost:5000 -u $username
Password: *******
Login Succeeded
```

\
