# Using OpenVPN for Samba

## OpenVPN Access Server

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

## Samba

https://www.bellmts.ca/support/internet/security/blocked-or-restricted-ports

TCP port 445 is blocked by my ISP. Using OpenVPN, we can use Samba on the standard port 445.

<img width="1088" height="487" alt="image" src="https://github.com/user-attachments/assets/76dee5a9-68e2-460c-a35d-339bd03b8d2e" />

We then see the file on the Samba server:

```bash
sudo stat /samba/sambauser/openvpn-connect-3.8.0.4528_signed.msi
```

```console
  File: /samba/sambauser/openvpn-connect-3.8.0.4528_signed.msi
  Size: 115945472       Blocks: 226464     IO Block: 4096   regular file
Device: 8,1     Inode: 1048580     Links: 1
Access: (0744/-rwxr--r--)  Uid: ( 1001/sambauser)   Gid: (  987/sambashare)
Access: 2025-12-09 04:35:32.246383259 +0000
Modify: 2025-12-09 03:58:07.502152800 +0000
Change: 2025-12-09 04:35:36.791463323 +0000
 Birth: 2025-12-09 04:35:32.246383259 +0000
```
