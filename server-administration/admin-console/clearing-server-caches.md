# Clearing Server Caches

Keycloak will cache everything it can in memory within the limits of your JVM and/or the limits you’ve configured it for. If the Keycloak database is modified by a third party (i.e. a DBA) outside the scope of the server’s REST APIs or Admin Console there’s a chance parts of the in-memory cache may be stale. You can clear the realm cache, user cache or cache of external public keys (Public keys of external clients or Identity providers, which Keycloak usually uses for verify signatures of particular external entity) from the Admin Console by going to the `Realm Settings` left menu item and the `Cache` tab.

Cache tab

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

Just click the `clear` button on the cache you want to evict.
