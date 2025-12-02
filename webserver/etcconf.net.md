# etcconf.net

https://www.digitalocean.com/community/tutorials/how-to-create-a-new-sudo-enabled-user-on-ubuntu

```console
root@lamp-ubuntu-s-2vcpu-4gb-nyc3-01:~# getent passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:996:996:systemd Time Synchronization:/:/usr/sbin/nologin
dhcpcd:x:100:65534:DHCP Client Daemon,,,:/usr/lib/dhcpcd:/bin/false
messagebus:x:101:101::/nonexistent:/usr/sbin/nologin
syslog:x:102:102::/nonexistent:/usr/sbin/nologin
systemd-resolve:x:991:991:systemd Resolver:/:/usr/sbin/nologin
uuidd:x:103:103::/run/uuidd:/usr/sbin/nologin
tss:x:104:104:TPM software stack,,,:/var/lib/tpm:/bin/false
sshd:x:105:65534::/run/sshd:/usr/sbin/nologin
pollinate:x:106:1::/var/cache/pollinate:/bin/false
tcpdump:x:107:108::/nonexistent:/usr/sbin/nologin
landscape:x:108:109::/var/lib/landscape:/usr/sbin/nologin
fwupd-refresh:x:990:990:Firmware update daemon:/var/lib/fwupd:/usr/sbin/nologin
polkitd:x:989:989:User for polkitd:/:/usr/sbin/nologin
mysql:x:109:112:MySQL Server,,,:/nonexistent:/bin/false
postfix:x:110:114::/var/spool/postfix:/usr/sbin/nologin
```

```console
root@lamp-ubuntu-s-2vcpu-4gb-nyc3-01:~# adduser ubuntu
info: Adding user `ubuntu' ...
info: Selecting UID/GID from range 1000 to 59999 ...
info: Adding new group `ubuntu' (1000) ...
info: Adding new user `ubuntu' (1000) with group `ubuntu (1000)' ...
info: Creating home directory `/home/ubuntu' ...
info: Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for ubuntu
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n]
info: Adding new user `ubuntu' to supplemental / extra groups `users' ...
info: Adding user `ubuntu' to group `users' ...
```

```bash
usermod -aG sudo ubuntu
```

https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-20-04


https://www.digitalocean.com/community/tutorials/how-to-install-lamp-stack-on-ubuntu

```console
mkdir -p /var/www/etcconf.net/public_html
```

```
vi /etc/apache2/sites-available/etcconf.net.conf
```

```
<VirtualHost *:80>
    ServerName etcconf.net
    ServerAlias www.etcconf.net
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/etcconf.net
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

```console
root@etcconf:~# vi /etc/apache2/sites-available/etcconf.net.conf
root@etcconf:~# sudo a2ensite etcconf.net
Enabling site etcconf.net.
To activate the new configuration, you need to run:
  systemctl reload apache2
```

<https://www.digitalocean.com/community/tutorials/apache-configuration-error-ah00558-could-not-reliably-determine-the-server-s-fully-qualified-domain-name#setting-a-global-servername-directive>

```bash
echo "ServerName 127.0.0.1" | tee -a /etc/apache2/apache2.conf
```



