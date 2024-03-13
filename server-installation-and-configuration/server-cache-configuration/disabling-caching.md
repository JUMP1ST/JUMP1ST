# Disabling Caching

To disable the realm or user cache, you must edit the `standalone.xml`, `standalone-ha.xml`, or `domain.xml` file in your distribution. The location of this file depends on your [operating mode](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/topics/operating-mode.html#\_operating-mode). Here’s what the config looks like initially.

```xml
    <spi name="userCache">
        <provider name="default" enabled="true"/>
    </spi>

    <spi name="realmCache">
        <provider name="default" enabled="true"/>
    </spi>
```

To disable the cache set the `enabled` attribute to false for the cache you want to disable. You must reboot your server for this change to take effect.
