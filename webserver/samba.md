# Samba

## Samba

https://www.samba.org/

https://www.digitalocean.com/community/tutorials/how-to-set-up-a-samba-share-for-a-small-organization-on-ubuntu-16-04

https://documentation.ubuntu.com/server/how-to/samba/file-server/

https://phoenixnap.com/kb/ubuntu-samba

```bash
sudo apt install samba samba-common
```

The network interfaces are found using

```bash
ip link
```

```console
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:44:c7:70 brd ff:ff:ff:ff:ff:ff
    altname enp0s3
```

namely, lo and ens3.


We edit the Samba config file

```bash
sudo vi /etc/samba/smb.conf
```

and place the following lines at the end

```
[share]
    comment = Samba on Ubuntu
    path = /samba/sambauser
    read only = no
    browsable = yes
    valid users = sambauser
```

We run

```bash
testparm
```

```console
Load smb config files from /etc/samba/smb.conf
Loaded services file OK.
Weak crypto is allowed by GnuTLS (e.g. NTLM as a compatibility fallback)

Server role: ROLE_STANDALONE

Press enter to see a dump of your service definitions

# Global parameters
[global]
        bind interfaces only = Yes
        interfaces = lo ens3
        log file = /var/log/samba/log.%m
        logging = file
        max log size = 1000
        panic action = /usr/share/samba/panic-action %d
        server role = standalone server
        server string = samba_server
        usershare allow guests = Yes
        idmap config * : backend = tdb


[share]
        comment = Samba on Ubuntu
        path = /samba/sambauser
        read only = No
        valid users = sambauser
```

Then we make a user "sambauser"

```bash
sudo adduser --home /samba/sambauser --no-create-home --shell /usr/sbin/nologin --ingroup sambashare sambauser
sudo chown sambauser:sambashare /samba/sambauser/
sudo chmod 2770 /samba/sambauser/
```

There is a separate Samba user, which we give the same username as the system user "sambauser". It has its own password distinct from
the system password.

```bash
sudo smbpasswd -a sambauser
sudo smbpasswd -e sambauser
```

```bash
sudo service smbd restart
```

On another machine,

```bash
smbclient //158.69.60.101/share -U sambauser
```

which opens the Samba client session, in which we upload a file "weather.sh"

```console
Password for [WORKGROUP\sambauser]:
Try "help" to get a list of possible commands.
smb: \> put weather.sh
putting file weather.sh as \weather.sh (137.1 kb/s) (average 137.1 kb/s)
smb: \> quit
```

On the Samba server,

```bash
sudo stat /samba/sambauser/weather.sh
```

```console
  File: /samba/sambauser/weather.sh
  Size: 702             Blocks: 8          IO Block: 4096   regular file
Device: 8,1     Inode: 1048579     Links: 1
Access: (0744/-rwxr--r--)  Uid: ( 1001/sambauser)   Gid: (  987/sambashare)
Access: 2025-12-07 03:34:05.419694560 +0000
Modify: 2025-12-07 03:33:03.297657669 +0000
Change: 2025-12-07 03:33:03.296621562 +0000
 Birth: 2025-12-07 03:33:03.293621510 +0000
```

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

## Using Samba with OpenVPN

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

