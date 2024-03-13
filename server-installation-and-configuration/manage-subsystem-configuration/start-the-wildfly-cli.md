# Start the Wildfly CLI

Besides editing the configuration by hand, you also have the option of changing the configuration by issuing commands via the _jboss-cli_ tool. CLI allows you to configure servers locally or remotely. And it is especially useful when combined with scripting.

To start the Wildfly CLI, you need to run `jboss-cli`.

Linux/Unix

```
$ .../bin/jboss-cli.sh
```

Windows

```
> ...\bin\jboss-cli.bat
```

This will bring you to a prompt like this:

Prompt

```
[disconnected /]
```

If you wish to execute commands on a running server, you will first execute the `connect` command.

connect

```
[disconnected /] connect
connect
[standalone@localhost:9990 /]
```

You may be thinking to yourself, "I didn’t enter in any username or password!". If you run `jboss-cli` on the same machine as your running standalone server or domain controller and your account has appropriate file permissions, you do not have to setup or enter in a admin username and password. See the [_WildFly 10 Documentation_](https://docs.jboss.org/author/display/WFLY10/Documentation) for more details on how to make things more secure if you are uncomfortable with that setup.

#### CLI Embedded Mode <a href="#cli_embedded_mode" id="cli_embedded_mode"></a>

If you do happen to be on the same machine as your standalone server and you want to issue commands while the server is not active, you can embed the server into CLI and make changes in a special mode that disallows incoming requests. To do this, first execute the `embed` command with the config file you wish to change.

embed

```
[disconnected /] embed-server --server-config=standalone.xml
[standalone@embedded /]
```

#### CLI GUI Mode <a href="#cli_gui_mode" id="cli_gui_mode"></a>

The CLI can also run in GUI mode. GUI mode launches a Swing application that allows you to graphically view and edit the entire management model of a _running_ server. GUI mode is especially useful when you need help formatting your CLI commands and learning about the options available. The GUI can also retrieve server logs from a local or remote server.

Start in GUI mode

```
$ .../bin/jboss-cli.sh --gui
```

_Note: to connect to a remote server, you pass the `--connect` option as well. Use the --help option for more details._

After launching GUI mode, you will probably want to scroll down to find the node, `subsystem=keycloak-server`. If you right-click on the node and click `Explore subsystem=keycloak-server`, you will get a new tab that shows only the keycloak-server subsystem.

![cli-gui.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_installation/images/cli-gui.png)

#### CLI Scripting <a href="#cli_scripting" id="cli_scripting"></a>

The CLI has extensive scripting capabilities. A script is just a text file with CLI commands in it. Consider a simple script that turns off theme and template caching.

turn-off-caching.cli

```
/subsystem=keycloak-server/theme=defaults/:write-attribute(name=cacheThemes,value=false)
/subsystem=keycloak-server/theme=defaults/:write-attribute(name=cacheTemplates,value=false)
```

To execute the script, I can follow the `Scripts` menu in CLI GUI, or execute the script from the command line as follows:

```
$ .../bin/jboss-cli.sh --file=turn-off-caching.cli
```
