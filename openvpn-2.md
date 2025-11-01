# OpenVPN Part 2

## Sign-up for OpenVPN and create Access Server

Sign-up for OpenVPN:

<https://openvpn.net/access-server/>

Deploy Access Server:

<https://as-portal.openvpn.com/instructions/digital-ocean/installation>

<https://openvpn.net/as-docs/digitalocean.html>

## Make local SSH key pair

On local machine:[^keygen]

[^keygen]: <https://www.ssh.com/academy/ssh/keygen>

```bash
ssh-keygen -t ed25519
cat .ssh/id_ed25519.pub
```

Add the public key to DigitalOcean

<https://docs.digitalocean.com/platform/teams/how-to/upload-ssh-keys/>

Connect to remote machine from local machine: https://docs.digitalocean.com/products/droplets/how-to/connect-with-ssh/openssh/

Then on local machine, login into remote machine as root user:

```
ssh -i ./.ssh/id_ed25519 root@167.99.112.225
```

Then make user: https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu

Execute following as root user:

```console
root@openvpnaccessserver2143onubuntu2404-s-1vcpu-2gb-nyc3-01:~# adduser openvpn
info: Adding user `openvpn' ...
info: Selecting UID/GID from range 1000 to 59999 ...
info: Adding new group `openvpn' (1001) ...
info: Adding new user `openvpn' (1001) with group `openvpn (1001)' ...
info: Creating home directory `/home/openvpn' ...
info: Copying files from `/etc/skel' ...
New password: 
Retype new password: 
passwd: password updated successfully
Changing the user information for openvpn
Enter the new value, or press ENTER for the default
        Full Name []: 
        Room Number []: 
        Work Phone []: 
        Home Phone []: 
        Other []: 
Is the information correct? [Y/n] Y
info: Adding new user `openvpn' to supplemental / extra groups `users' ...
info: Adding user `openvpn' to group `users' ...
```

```console
root@openvpnaccessserver2143onubuntu2404-s-1vcpu-2gb-nyc3-01:~# usermod -aG sudo openvpn
```

```console
root@openvpnaccessserver2143onubuntu2404-s-1vcpu-2gb-nyc3-01:~# ufw allow OpenSSH
Rules updated
Rules updated (v6)
root@openvpnaccessserver2143onubuntu2404-s-1vcpu-2gb-nyc3-01:~# ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```

> The simplest way to copy the files with the correct ownership and permissions is with the rsync command. This command will copy the root user’s .ssh directory, preserve the permissions, and modify the file owners, all in a single command. Make sure to change the highlighted portions of the command below to match your regular user’s name:

```console
root@openvpnaccessserver2143onubuntu2404-s-1vcpu-2gb-nyc3-01:~# rsync --archive --chown=openvpn:openvpn ~/.ssh /home/openvpn
```

Then on local machine, connect to remote machine as user openvpn, using the private key file we made and used already:

```console
ubuntu@LAPTOP-JBell:~$ ssh -i ./.ssh/id_ed25519 openvpn@165.227.105.57
Welcome to OpenVPN Access Server Appliance 2.14.3

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Fri Oct 31 02:52:42 UTC 2025

  System load:           0.01
  Usage of /:            5.8% of 47.39GB
  Memory usage:          28%
  Swap usage:            0%
  Processes:             117
  Users logged in:       1
  IPv4 address for eth0: 165.227.105.57
  IPv4 address for eth0: 10.17.0.5
  IPv6 address for eth0: 2604:a880:800:14:0:1:ecd1:9000

Expanded Security Maintenance for Applications is not enabled.

30 updates can be applied immediately.
1 of these updates is a standard security update.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


*** System restart required ***
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

openvpn@openvpnaccessserver2143onubuntu2404-s-1vcpu-2gb-nyc3-01:~$
```


## Step 3

https://openvpn.net/as-docs/tutorials/tutorial--custom-hostname.html#overview-104605


# Step 4

https://openvpn.net/as-docs/tutorials/tutorial--connect-with-linux.html#option-1--connecting-with-the-openvpn-3-linux-client--using-openvpn3-as-to-download-a-profile-

https://support.openvpn.com/hc/en-us/articles/19342886268059-Access-Server-Import-a-connection-profile-ovpn-file-directly-using-OpenVPN3-Linux-client-via-openvpn3-as-tool

Local machine:

```console
ubuntu@LAPTOP-JBell:~$ openvpn3-as --name openvpn.ovpn https://openvpn.jordanbell.org/ --insecure-certs
OpenVPN Access Server Username: openvpn
OpenVPN Access Server Password:
------------------------------------------------------------
Profile imported successfully
Configuration name: openvpn.ovpn
Configuration path: /net/openvpn/v3/configuration/3b68e4d9xeedfx4c22xba91x63d63baa61ce
```

```console
ubuntu@LAPTOP-JBell:~$ openvpn3 session-start --config openvpn.ovpn
Using pre-loaded configuration profile 'openvpn.ovpn'
Session path: /net/openvpn/v3/sessions/69f95e3dsa9fes4cf4s8645s35de1912e170
Auth User name: openvpn
Auth Password:
Connected to 165.227.105.57 (openvpn.jordanbell.org)
```

```bash
openssl req -out server.csr -new -newkey rsa:4096 -sha256 -nodes -keyout server.key
```

```
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:NewYork
Locality Name (eg, city) []:New York City
Organization Name (eg, company) [Internet Widgits Pty Ltd]:dev
Organizational Unit Name (eg, section) []:dev
Common Name (e.g. server FQDN or YOUR name) []:openvpn.jordanbell.info
Email Address []:jordan.bell@gmail.com
```

```console
ubuntu@LAPTOP-JBell:~$ curl https://api.my-ip.io/v2/ip.txt
165.227.105.57
IPv4
US
United States
New Jersey
Clifton
40.8364
-74.1403
America/New_York
14061
DIGITALOCEAN-ASN
165.227.0.0/16
```

```console
ubuntu@LAPTOP-JBell:~$ openvpn3 session-manage -c openvpn.ovpn -D
Initiated session shutdown.

Connection statistics:
     BYTES_IN..................100266
     BYTES_OUT............25285815248
     PACKETS_IN...................473
     PACKETS_OUT.............41191246
     TUN_BYTES_IN.........24297220649
     TUN_BYTES_OUT..............82684
     TUN_PACKETS_IN..........41191204
     TUN_PACKETS_OUT..............409
     NETWORK_SEND_ERROR.............3
     KEEPALIVE_TIMEOUT..............1
     N_RECONNECT....................1
```

```console
ubuntu@LAPTOP-JBell:~$ curl https://api.my-ip.io/v2/ip.txt
70.55.4.185
IPv4
CA
Canada
Ontario
Toronto
43.6508
-79.4803
America/Toronto
577
BACOM
70.55.0.0/18
```

## Step 5

https://openvpn.net/as-docs/tutorials/tutorial--install-ssl-certificate.html
