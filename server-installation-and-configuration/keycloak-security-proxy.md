# Keycloak Security Proxy

Keycloak has an HTTP(S) proxy that you can put in front of web applications and services where it is not possible to install the Keycloak adapter. You can set up URL filters so that certain URLs are secured either by browser login and/or bearer token authentication. You can also define role constraints for URL patterns within your applications.

#### Proxy Install and Run <a href="#proxy_install_and_run" id="proxy_install_and_run"></a>

Download the Keycloak proxy distribution from the Keycloak download pages and unzip it.

```
$ unzip keycloak-proxy-dist.zip
```

To run it you must have a proxy config file (which we’ll discuss in a moment).

```
$ java -jar bin/launcher.jar [your-config.json]
```

If you do not specify a path to the proxy config file, the launcher will look in the current working directory for the file named `proxy.json`

#### Proxy Configuration <a href="#proxy_configuration" id="proxy_configuration"></a>

Here’s an example configuration file.

```
{
    "target-url": "http://localhost:8082",
    "send-access-token": true,
    "bind-address": "localhost",
    "http-port": "8080",
    "https-port": "8443",
    "keystore": "classpath:ssl.jks",
    "keystore-password": "password",
    "key-password": "password",
    "applications": [
        {
            "base-path": "/customer-portal",
            "error-page": "/error.html",
            "adapter-config": {
                "realm": "demo",
                "resource": "customer-portal",
                "realm-public-key": "MIGfMA0GCSqGSIb",
                "auth-server-url": "http://localhost:8081/auth",
                "ssl-required" : "external",
                "principal-attribute": "name",
                "credentials": {
                    "secret": "password"
                }
            }
            ,
            "constraints": [
                {
                    "pattern": "/users/*",
                    "roles-allowed": [
                        "user"
                    ]
                },
                {
                    "pattern": "/admins/*",
                    "roles-allowed": [
                        "admin"
                    ]
                },
                {
                    "pattern": "/users/permit",
                    "permit": true
                },
                {
                    "pattern": "/users/deny",
                    "deny": true
                }
            ]
        }
    ]
}
```

**Basic Config**

The basic configuration options for the server are as follows:

target-url

The URL this server is proxying _REQUIRED._.

send-access-token

Boolean flag. If true, this will send the access token via the KEYCLOAK\_ACCESS\_TOKEN header to the proxied server. _OPTIONAL._. Default is false.

bind-address

DNS name or IP address to bind the proxy server’s sockets to. _OPTIONAL._. The default value is _localhost_

http-port

Port to listen for HTTP requests. If you do not specify this value, then the proxy will not listen for regular HTTP requests. _OPTIONAL._.

https-port

Port to listen for HTTPS requests. If you do not specify this value, then the proxy will not listen for HTTPS requests. _OPTIONAL._.

keystore

Path to a Java keystore file that contains private key and certificate for the server to be able to handle HTTPS requests. Can be a file path, or, if you prefix it with `classpath:` it will look for this file in the classpath. _OPTIONAL._. If you have enabled HTTPS, but have not defined a keystore, the proxy will auto-generate a self-signed certificate and use that.

buffer-size

HTTP server socket buffer size. Usually the default is good enough. _OPTIONAL._.

buffers-per-region

HTTP server socket buffers per region. Usually the default is good enough. _OPTIONAL._.

io-threads

Number of threads to handle IO. Usually default is good enough. _OPTIONAL._. The default is the number of available processors \* 2.

worker-threads

Number of threads to handle requests. Usually the default is good enough. _OPTIONAL._. The default is the number of available processors \* 16.

#### Application Config <a href="#application_config" id="application_config"></a>

Next under the `applications` array attribute, you can define one or more applications per host you are proxying.

base-path

The base context root for the application. Must start with '/' _REQUIRED._.

error-page

If the proxy has an error, it will display the target application’s error page relative URL _OPTIONAL._. This is a relative path to the base-path. In the example above it would be `/customer-portal/error.html`.

adapter-config

_REQUIRED._. Same configuration as any other Keycloak adapter. // See [Adapter Config](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/proxy.html#\_adapter\_config)

**Constraint Config**

Next under each application you can define one or more constraints in the `constraints` array attribute. A constraint defines a URL pattern relative to the base-path. You can deny, permit, or require authentication for a specific URL pattern. You can specify roles allowed for that path as well. More specific constraints will take precedence over more general ones.

pattern

URL pattern to match relative to the base-path of the application. Must start with '/' _REQUIRED._ You may only have one wildcard and it must come at the end of the pattern.

* Valid: `/foo/bar/*` and `/foo/*.txt`
* Not valid: `/*/foo/*`.

roles-allowed

Array of strings of roles allowed to access this url pattern. _OPTIONAL._.

methods

Array of strings of HTTP methods that will exclusively match this pattern and HTTP request. _OPTIONAL._.

excluded-methods

Array of strings of HTTP methods that will be ignored when match this pattern. _OPTIONAL._.

deny

Deny all access to this URL pattern. _OPTIONAL._.

permit

Permit all access without requiring authentication or a role mapping. _OPTIONAL._.

permit-and-inject

Permit all access, but inject the headers, if user is already authenticated._OPTIONAL._.

authenticate

Require authentication for this pattern, but no role mapping. _OPTIONAL._.

**Header Names Config**

Next under the list of applications you can override the defaults for the names of the header fields injected by the proxy (see Keycloak Identity Headers). This mapping is optional.

keycloak-subject

e.g. MYAPP\_USER\_ID

keycloak-username

e.g. MYAPP\_USER\_NAME

keycloak-email

e.g. MYAPP\_USER\_EMAIL

keycloak-name

e.g. MYAPP\_USER\_ID

keycloak-access-token

e.g. MYAPP\_ACCESS\_TOKEN

#### Keycloak Identity Headers <a href="#keycloak_identity_headers" id="keycloak_identity_headers"></a>

When forwarding requests to the proxied server, Keycloak Proxy will set some additional headers with values from the OIDC identity token it received for authentication.

KEYCLOAK\_SUBJECT

User id. Corresponds to JWT `sub` and will be the user id Keycloak uses to store this user.

KEYCLOAK\_USERNAME

Username. Corresponds to JWT `preferred_username`

KEYCLOAK\_EMAIL

Email address of user if set.

KEYCLOAK\_NAME

Full name of user if set.

KEYCLOAK\_ACCESS\_TOKEN

Send the access token in this header if the proxy was configured to send it. This token can be used to make bearer token requests. Header field names can be configured using a map of `header-names` in configuration file:

```
{
    "header-names" {
        "keycloak-subject": "MY_SUBJECT"
    }
}
```
