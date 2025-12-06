# Sambda

https://www.digitalocean.com/community/tutorials/how-to-set-up-a-samba-share-for-a-small-organization-on-ubuntu-16-04

https://documentation.ubuntu.com/server/how-to/samba/file-server/

https://phoenixnap.com/kb/ubuntu-samba

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


```bash
sudo vi /etc/samba/smb.conf
```

```
[global]
        server string = sambda.histfile.org
        server role = standalone server
        interfaces = lo ens3
        bind interfaces only = yes
        disable netbios = yes
        smb ports = 445
        log file = /var/log/samba/smb.log
        max log size = 10000
```

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
        disable netbios = Yes
        interfaces = lo ens3
        log file = /var/log/samba/smb.log
        max log size = 10000
        server role = standalone server
        server string = sambda.histfile.org
        smb ports = 445
        idmap config * : backend = tdb
```

```bash
sudo mkdir -p /samba/sambauser
sudo chown :sambashare -R /samba/
sudo adduser --home /samba/sambauser --no-create-home --shell /usr/sbin/nologin --ingroup sambashare sambauser
```

```bash
sudo chown sambauser:sambashare /samba/sambauser/
sudo chmod 2770 /samba/sambauser/
```

```bash
sudo smbpasswd -a sambauser
sudo smbpasswd -e sambauser
```

```bash
sudo vi /etc/sambda/smb.conf
```

```console
[sambauser]
        path = /samba/sambauser
        browseable = no
        read only = no
        force create mode = 0660
        force directory mode = 2770
        valid users = sambauser
```
