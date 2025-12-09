# OpenVPN Access Server

https://as-portal.openvpn.com/quick-start

Connect using root SSH and configure OpenVPN Access Server. During configuration, we enter the Activation
Key for Access Server. 

The configuration ends with the following:

```console
Initial Configuration Complete!

You can now continue configuring OpenVPN Access Server by
directing your Web browser to this URL:

https://64.225.5.143:943/admin

During normal operation, OpenVPN AS can be accessed via these URLs:
Admin  UI: https://64.225.5.143:943/admin
Client UI: https://64.225.5.143:943/
To login please use the "openvpn" account with the password you specified during the setup.

See the Release Notes for this release at:
   https://openvpn.net/vpn-server-resources/release-notes/
```

We download and install OpenVPN Connect client.

https://openvpn.net/client/

We then download the Connection Profile from

https://64.225.5.143:943/

profile-userlocked.ovpn
