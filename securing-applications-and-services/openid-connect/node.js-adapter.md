# Node.js Adapter

Keycloak provides a Node.js adapter built on top of [Connect](https://github.com/senchalabs/connect) to protect server-side JavaScript apps — the goal was to be flexible enough to integrate with frameworks like [Express.js](https://expressjs.com/).

The library can be downloaded directly from [Keycloak organization](https://www.npmjs.com/package/keycloak-connect) and the source is available at [GitHub](https://github.com/keycloak/keycloak-nodejs-connect).

To use the Node.js adapter, first you must create a client for your application in the Keycloak Administration Console. The adapter supports public, confidential, and bearer-only access type. Which one to choose depends on the use-case scenario.

Once the client is created click the `Installation` tab, select `Keycloak OIDC JSON` for `Format Option`, and then click `Download`. The downloaded `keycloak.json` file should be at the root folder of your project.

**Installation**

Assuming you’ve already installed [Node.js](https://nodejs.org/), create a folder for your application:

```
mkdir myapp && cd myapp
```

Use `npm init` command to create a `package.json` for your application. Now add the Keycloak connect adapter in the dependencies list:

```json
    "dependencies": {
        "keycloak-connect": "SNAPSHOT"
    }
```

**Usage**

Instantiate a Keycloak class

The `Keycloak` class provides a central point for configuration and integration with your application. The simplest creation involves no arguments.

```javascript
    var session = require('express-session');
    var Keycloak = require('keycloak-connect');

    var memoryStore = new session.MemoryStore();
    var keycloak = new Keycloak({ store: memoryStore });
```

By default, this will locate a file named `keycloak.json` alongside the main executable of your application to initialize keycloak-specific settings (public key, realm name, various URLs). The `keycloak.json` file is obtained from the Keycloak Admin Console.

Instantiation with this method results in all of the reasonable defaults being used. As alternative, it’s also possible to provide a configuration object, rather than the `keycloak.json` file:

```javascript
    let kcConfig = {
    clientId: 'myclient',
    bearerOnly: true,
    serverUrl: 'http://localhost:8080/auth',
    realm: 'myrealm',
    realmPublicKey: 'MIIBIjANB...'
};

let keycloak = new Keycloak({ store: memoryStore }, kcConfig);
```

Configuring a web session store

If you want to use web sessions to manage server-side state for authentication, you need to initialize the `Keycloak(…​)` with at least a `store` parameter, passing in the actual session store that `express-session` is using.

```javascript
    var session = require('express-session');
    var memoryStore = new session.MemoryStore();

    var keycloak = new Keycloak({ store: memoryStore });
```

Passing a custom scope value

By default, the scope value `openid` is passed as a query parameter to Keycloak’s login URL, but you can add an additional custom value:

```javascript
    var keycloak = new Keycloak({ scope: 'offline_access' });
```

**Installing Middleware**

Once instantiated, install the middleware into your connect-capable app:

```javascript
    var app = express();

    app.use( keycloak.middleware() );
```

**Protecting Resources**

Simple authentication

To enforce that an user must be authenticated before accessing a resource, simply use a no-argument version of `keycloak.protect()`:

```javascript
    app.get( '/complain', keycloak.protect(), complaintHandler );
```

Role-based authorization

To secure a resource with an application role for the current app:

```javascript
    app.get( '/special', keycloak.protect('special'), specialHandler );
```

To secure a resource with an application role for a **different** app:

```javascript
    app.get( '/extra-special', keycloak.protect('other-app:special', extraSpecialHandler );
```

To secure a resource with a realm role:

```javascript
    app.get( '/admin', keycloak.protect( 'realm:admin' ), adminHandler );
```

Advanced authorization

To secure resources based on parts of the URL itself, assuming a role exists for each section:

```javascript
    function protectBySection(token, request) {
      return token.hasRole( request.params.section );
    }

    app.get( '/:section/:page', keycloak.protect( protectBySection ), sectionHandler );
```

**Additional URLs**

Explicit user-triggered logout

By default, the middleware catches calls to `/logout` to send the user through a Keycloak-centric logout workflow. This can be changed by specifying a `logout` configuration parameter to the `middleware()` call:

```javascript
    app.use( keycloak.middleware( { logout: '/logoff' } ));
```

Keycloak Admin Callbacks

Also, the middleware supports callbacks from the Keycloak console to log out a single session or all sessions. By default, these type of admin callbacks occur relative to the root URL of `/` but can be changed by providing an `admin` parameter to the `middleware()` call:

```javascript
    app.use( keycloak.middleware( { admin: '/callbacks' } );
```
