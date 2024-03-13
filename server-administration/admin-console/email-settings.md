# Email Settings

Keycloak sends emails to users to verify their email address, when they forget their passwords, or when an admin needs to receive notifications about a server event. To enable Keycloak to send emails you need to provide Keycloak with your SMTP server settings. This is configured per realm. Go to the `Realm Settings` left menu item and click the `Email` tab.

Email Tab

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

Host

`Host` denotes the SMTP server hostname used for sending emails.

Port

`Port` denotes the SMTP server port.

From

`From` denotes the address used for the `From` SMTP-Header for the emails sent.

From Display Name

`From Display Name` allows to configure an user friendly email address aliases (optional). If not set the plain `From` email address will be displayed in email clients.

Reply To

`Reply To` denotes the address used for the `Reply-To` SMTP-Header for the mails sent (optional). If not set the plain `From` email address will be used.

Reply To Display Name

`Reply To Display Name` allows to configure an user friendly email address aliases (optional). If not set the plain `Reply To` email address will be displayed.

Envelope From

`Envelope From` denotes the [Bounce Address](https://en.wikipedia.org/wiki/Bounce\_address) used for the `Return-Path` SMTP-Header for the mails sent (optional).

As emails are used for recovering usernames and passwords it’s recommended to use SSL or TLS, especially if the SMTP server is on an external network. To enable SSL click on `Enable SSL` or to enable TLS click on `Enable TLS`. You will most likely also need to change the `Port` (the default port for SSL/TLS is 465).

If your SMTP server requires authentication click on `Enable Authentication` and insert the `Username` and `Password`.
