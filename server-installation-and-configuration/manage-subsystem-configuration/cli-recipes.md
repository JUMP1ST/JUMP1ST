# CLI Recipes

Here are some configuration tasks and how to perform them with CLI commands. Note that in all but the first example, we use the wildcard path `**` to mean you should substitute or the path to the keycloak-server subsystem.

For standalone, this just means:

`**` = `/subsystem=keycloak-server`

For domain mode, this would mean something like:

`**` = `/profile=auth-server-clustered/subsystem=keycloak-server`

**Change the web context of the server**

```
/subsystem=keycloak-server/:write-attribute(name=web-context,value=myContext)
```

**Set the global default theme**

```
**/theme=defaults/:write-attribute(name=default,value=myTheme)
```

**Add a new SPI and a provider**

```
**/spi=mySPI/:add
**/spi=mySPI/provider=myProvider/:add(enabled=true)
```

**Disable a provider**

```
**/spi=mySPI/provider=myProvider/:write-attribute(name=enabled,value=false)
```

**Change the default provider for an SPI**

```
**/spi=mySPI/:write-attribute(name=default-provider,value=myProvider)
```

**Configure the dblock SPI**

```
**/spi=dblock/:add(default-provider=jpa)
**/spi=dblock/provider=jpa/:add(properties={lockWaitTimeout => "900"},enabled=true)
```

**Add or change a single property value for a provider**

```
**/spi=dblock/provider=jpa/:map-put(name=properties,key=lockWaitTimeout,value=3)
```

**Remove a single property from a provider**

```
**/spi=dblock/provider=jpa/:map-remove(name=properties,key=lockRecheckTime)
```

**Set values on a provider property of type `List`**

```
**/spi=eventsStore/provider=jpa/:map-put(name=properties,key=exclude-events,value=[EVENT1,EVENT2])
```
