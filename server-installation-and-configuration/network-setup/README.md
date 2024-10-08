# Network Setup

Keycloak can run out of the box with some networking limitations. For one, all network endpoints bind to `localhost` so the auth server is really only usable on one local machine. For HTTP based connections, it does not use default ports like 80 and 443. HTTPS/SSL is not configured out of the box and without it, Keycloak has many security vulnerabilities. Finally, Keycloak may often need to make secure SSL and HTTPS connections to external servers and thus need a trust store set up so that endpoints can be validated correctly. This chapter discusses all of these things.
