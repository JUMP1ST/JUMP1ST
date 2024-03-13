# SSSD and FreeIPA Identity Management Integration

Keycloak also comes with a built-in [SSSD](https://fedorahosted.org/sssd/wiki) (System Security Services Daemon) plugin. SSSD is part of the latest Fedora or Red Hat Enterprise Linux and provides access to multiple identity and authentication providers. It provides benefits such as failover and offline support. To see configuration options and for more information see [the Red Hat Enterprise Linux Identity Management documentation](https://access.redhat.com/documentation/en-US/Red\_Hat\_Enterprise\_Linux/7/html/System-Level\_Authentication\_Guide/SSSD.html).

SSSD also integrates with the [FreeIPA identity management (IdM)](http://www.freeipa.org/page/Main\_Page) server, providing authentication and access control. For {book\_project\_name}, we benefit from this integration authenticating against [PAM](http://tldp.org/HOWTO/User-Authentication-HOWTO/x115.html) services and retrieving user data from SSSD. For more information about using Red Hat Identity Management in Linux environments, see [the Red Hat Enterprise Linux Identity Management documentation](https://access.redhat.com/documentation/en-US/Red\_Hat\_Enterprise\_Linux/7/html/Linux\_Domain\_Identity\_Authentication\_and\_Policy\_Guide/index.html).

![keycloak-sssd-freeipa-integration-overview.png](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_admin/keycloak-images/keycloak-sssd-freeipa-integration-overview.png)

Most of the communication between Keycloak and SSSD occurs through read-only D-Bus interfaces. For this reason, the only way to provision and update users is to use the FreeIPA/IdM administration interface. By default, like the LDAP federation provider, it is set up only to import username, email, first name, and last name.

| Note | Groups and roles are automatically registered, but not synchronized, so any changes made by the Keycloak administrator directly in Keycloak is not synchronized with SSSD. |
| ---- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

Information on how to configure the FreeIPA/IdM server follows.

**FreeIPA/IdM Server**

As a matter of simplicity, a [FreeIPA Docker image](https://www.freeipa.org/page/Docker) already available is used. To set up a server, see the [FreeIPA documentation](https://www.freeipa.org/page/Quick\_Start\_Guide).

Running a FreeIPA server with Docker requires this command:

```
docker run --name freeipa-server-container -it \
-h server.freeipa.local -e PASSWORD=YOUR_PASSWORD \
-v /sys/fs/cgroup:/sys/fs/cgroup:ro \
-v /var/lib/ipa-data:/data:Z adelton/freeipa-server
```

The parameter `-h` with `server.freeipa.local` represents the FreeIPA/IdM server hostname. Be sure to change `YOUR_PASSWORD` to a password of your choosing.

After the container starts, change `/etc/hosts` to:

```
x.x.x.x     server.freeipa.local
```

If you do not make this change, you must set up a DNS server.

So that the SSSD federation provider is started and running on Keycloak you must enroll your Linux machine in the IPA domain:

```
ipa-client-install --mkhomedir -p admin -w password
```

To ensure that everything is working as expected, on the client machine, run:

```
kinit admin
```

You should be prompted for the password. After that, you can add users to the IPA server using this command:

```
$ ipa user-add john --first=John --last=Smith  --phone=042424242 --street="Testing street" \      --city="Testing city" --state="Testing State" --postalcode=0000000000
```

**SSSD and D-Bus**

As mentioned previously, the federation provider obtains the data from SSSD using D-BUS and authentication occurs using [PAM](http://tldp.org/HOWTO/User-Authentication-HOWTO/x115.html).

First, you have to install the sssd-dbus RPM, which allows information from SSSD to be transmitted over the system bus.

```
$ sudo yum install sssd-dbus
```

You must run the provisioning script available from the Keycloak distribution:

```
$ bin/federation-sssd-setup.sh
```

This script makes the necessary changes to `/etc/sssd/sssd.conf`:

```
[domain/your-hostname.local]
...
ldap_user_extra_attrs = mail:mail, sn:sn, givenname:givenname, telephoneNumber:telephoneNumber
...
[sssd]
services = nss, sudo, pam, ssh, ifp
...
[ifp]
allowed_uids = root, yourOSUsername
user_attributes = +mail, +telephoneNumber, +givenname, +sn
```

Also, a `keycloak` file is included under `/etc/pam.d/`:

```
auth    required   pam_sss.so
account required   pam_sss.so
```

Ensure everything is working as expected by running `dbus-send`:

```
sudo dbus-send --print-reply --system --dest=org.freedesktop.sssd.infopipe /org/freedesktop/sssd/infopipe org.freedesktop.sssd.infopipe.GetUserGroups string:john
```

You should be able to see the user’s group. If this command returns a timeout or an error, it means that the federation provider will also not be able to retrieve anything on Keycloak.

Most of the time this occurs because the machine was not enrolled in the FreeIPA IdM server or you do not have permission to access the SSSD service.

If you do not have permission, ensure that the user running Keycloak is included in the `/etc/sssd/sssd.conf` file in the following section:

```
[ifp]
allowed_uids = root, your_username
```

**Enabling the SSSD Federation Provider**

Keycloak uses DBus-Java to communicate at a low level with D-Bus, which depends on the [Unix Sockets Library](http://www.matthew.ath.cx/projects/java/).

An RPM for this library can be found in [this repository](https://github.com/keycloak/libunix-dbus-java/releases). Before installing it, be sure to check the RPM signature:

```
$ rpm -K libunix-dbus-java-0.8.0-1.fc24.x86_64.rpm
libunix-dbus-java-0.8.0-1.fc24.x86_64.rpm:
  Header V4 RSA/SHA256 Signature, key ID 84dc9914: OK
  Header SHA1 digest: OK (d17bb7ebaa7a5304c1856ee4357c8ba4ec9c0b89)
  V4 RSA/SHA256 Signature, key ID 84dc9914: OK
  MD5 digest: OK (770c2e68d052cb4a4473e1e9fd8818cf)
$ sudo yum install libunix-dbus-java-0.8.0-1.fc24.x86_64.rpm
```

For authentication with PAM Keycloak uses JNA. Be sure you have this package installed:

```
$ sudo yum install jna
```

#### Configuring a Federated SSSD Store <a href="#configuring_a_federated_sssd_store" id="configuring_a_federated_sssd_store"></a>

After installation, you need to configure a federated SSSD store.

To configure a federated SSSD store, complete the following steps:

1. Navigate to the Administration Console.
2. From the left menu, select **User Federation.**
3. From the **Add Provider** dropdown list, select **sssd.** The sssd configuration page opens.
4. Click **Save**.

Now you can authenticate against Keycloak using FreeIPA/IdM credentials.
