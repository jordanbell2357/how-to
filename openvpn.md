# OpenVPN

## Introduction

L = local Linux machine (WSL)

S = remote Linux machine (OpenVPN Access Server)

## Make local SSH key pairs on local machine L

> SSH provides an authentication mechanism based on cryptographic keys, called public key authentication.
> One or more public keys may be configured as authorized keys; the private key corresponding to an authorized key serves as authentication to the server.
> Typically both authorized keys and private keys are stored in the .ssh directory in a user's home directory.
> Fundamentally, such keys are like fancy passwords, only the password cannot be stolen from the network and it is possible to encrypt the private key locally
> (so that using it requires both a file and a passphrase only known to a user). However, in practice most keys are used for automation and do not have a passphrase. [^ssh]

[^ssh]: <https://www.ssh.com/academy/ssh/protocol>

We make an SSH key pairs using key file `OPEN_VPN`. [^ssh-key-gen]

[^ssh-key-gen]: <https://www.ssh.com/academy/ssh/keygen>

```console
ubuntu@LAPTOP-JBell:~$ ssh-keygen -t ed25519 -C "OPEN_VPN" -f ~/.ssh/OPEN_VPN
Generating public/private ed25519 key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ubuntu/.ssh/OPEN_VPN
Your public key has been saved in /home/ubuntu/.ssh/OPEN_VPN.pub

ubuntu@LAPTOP-JBell:~$ cat .ssh/OPEN_VPN.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFwJ1Lw8LwnsmZZRd0AQ5arvqfNqZ0Y59wm9vtdozZiH OPEN_VPN
```

## Add SSH public key OPEN_VPN.pub to DigitalOcean account

We then add the public key OPEN_VPN.pub to DigitalOcean and will associate the key with a droplet, following
<https://docs.digitalocean.com/platform/teams/how-to/upload-ssh-keys/>

## Create OpenVPN account

<https://openvpn.net/access-server/>

## Deploy Access Server S

Associate SSH public key OPEN_VPN.pub to droplet.

<https://as-portal.openvpn.com/instructions/digital-ocean/installation>

<https://openvpn.net/as-docs/digitalocean.html>

## Connect from L to S as root user using SSH private key OPEN_VPN and activate

Now we use SSH to connect from L to system S. First we activate the OpenVPN Access Server S by accessing it with SSH, and using the
activation key in <https://as-portal.openvpn.com/instructions/digital-ocean/activation>

We make a user `openvpn`.

```console
ubuntu@LAPTOP-JBell:~$ ssh -i .ssh/OPEN_VPN -l root 159.65.255.114
The authenticity of host '159.203.164.175 (159.203.164.175)' can't be established.
ED25519 key fingerprint is SHA256:RopZkdq6IqhhticLRdj41aVtPgsu/uwig5NyO1iYny8.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '159.203.164.175' (ED25519) to the list of known hosts.
Welcome to OpenVPN Access Server Appliance 2.14.3
⋮
```

We agree to the agreement, and then use default choices. Then we use our activation key ACTIVATION_KEY from
<https://as-portal.openvpn.com/instructions/digital-ocean/activation>

```console
To initially login to the Admin Web UI, you must use a
username and password that successfully authenticates you
with the host UNIX system (you can later modify the settings
so that RADIUS or LDAP is used for authentication instead).

You can login to the Admin Web UI as "openvpn" or specify
a different user account to use for this purpose.

Do you wish to login to the Admin UI as "openvpn"?
> Press ENTER for default [yes]:
Type a password for the 'openvpn' account (if left blank, a random password will be generated):
Confirm the password for the 'openvpn' account:

> Please specify your Activation key (or leave blank to specify later): ACTIVATION_KEY
Activation succeeded
```

Then

```console
You can login to the Admin Web UI as "openvpn" or specify
a different user account to use for this purpose.

Do you wish to login to the Admin UI as "openvpn"?
> Press ENTER for default [yes]:
Type a password for the 'openvpn' account (if left blank, a random password will be generated):
Confirm the password for the 'openvpn' account:
```

Last

```console
Initial Configuration Complete!

You can now continue configuring OpenVPN Access Server by
directing your Web browser to this URL:

https://159.65.255.114:943/admin

During normal operation, OpenVPN AS can be accessed via these URLs:
Admin  UI: https://159.65.255.114:943/admin
Client UI: https://159.65.255.114:943/
To login please use the "openvpn" account with the password you specified during the setup.
```

We then use a browser to access the OpenVPN Access Server Admin UI at <https://159.65.255.114:943/admin>, and agree to the
terms of service.

Following <https://openvpn.net/as-docs/digitalocean.html>, we do

```console
openvpn@ASBuildImage-ubuntu24-v2:~$ sudo dpkg-reconfigure tzdata
[sudo] password for openvpn:

Current default time zone: 'America/New_York'
Local time is now:      Sat Nov  1 11:00:11 EDT 2025.
Universal Time is now:  Sat Nov  1 15:00:11 UTC 2025.
```

## Make non-root user on OpenVPN Access Server S and give private key OPEN_VPN

We now make a non-root user `openvpn` on machine S.

```console
root@ASBuildImage-ubuntu24-v2:~# adduser openvpn
```

```console
root@ASBuildImage-ubuntu24-v2:~# usermod -aG sudo openvpn
```

We have been able to access S from L because the public key OPEN_VPN.pub is installed on droplet creation in the authorized_keys file on machine S: [^authorized-keys]

[^authorized-keys]: <https://www.ssh.com/academy/ssh/authorized-keys-openssh>

```console
root@ASBuildImage-ubuntu24-v2:~# cat /etc/ssh/sshd_config | grep AuthorizedKeysFile
#AuthorizedKeysFile     .ssh/authorized_keys .ssh/authorized_keys2
root@ASBuildImage-ubuntu24-v2:~# cat .ssh/authorized_keys
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFwJ1Lw8LwnsmZZRd0AQ5arvqfNqZ0Y59wm9vtdozZiH OPEN_VPN
```

We add the public key OPEN_VPN.pub to the authorized_keys file of user `openvpn`.

Then we  use rsync to copy the SSH key pair from the root user home folder of S to the `openvpn` user home folder of S: [^rsync]

[^rsync]: <https://download.samba.org/pub/rsync/rsync.1>

```console
root@EASY-RSA-s-1vcpu-1gb-35gb-intel-sfo2-01:~# rsync --archive --chown=openvpn:openvpn ~/.ssh /home/openvpn
root@ASBuildImage-ubuntu24-v2:~# cat /home/openvpn/.ssh/authorized_keys
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFwJ1Lw8LwnsmZZRd0AQ5arvqfNqZ0Y59wm9vtdozZiH OPEN_VPN
```

## ufw on S

```console
openvpn@ASBuildImage-ubuntu24-v2:~/EasyRSA-3.2.4$ sudo ufw app list
Available applications:
  OpenSSH
openvpn@ASBuildImage-ubuntu24-v2:~/EasyRSA-3.2.4$ sudo ufw allow OpenSSH
Rules updated
Rules updated (v6)
openvpn@ASBuildImage-ubuntu24-v2:~/EasyRSA-3.2.4$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```

https://openvpn.net/community-docs/openvpn-client-for-linux.html

## openvpn3-as on L

<https://support.openvpn.com/hc/en-us/articles/19342886268059-Access-Server-Import-a-connection-profile-ovpn-file-directly-using-OpenVPN3-Linux-client-via-openvpn3-as-tool>

```console
ubuntu@LAPTOP-JBell:~$ openvpn3-as --insecure-certs --username openvpn https://159.65.255.114/
OpenVPN Access Server Password:
------------------------------------------------------------
Profile imported successfully
Configuration name: AS:159.65.255.114
Configuration path: /net/openvpn/v3/configuration/54c3da09xf05bx492exbd55x177e0f882848
```


```console
ubuntu@LAPTOP-JBell:~$ openvpn3 configs-list
Configuration Name                                        Last used
------------------------------------------------------------------------------
openvpn.ovpn                                              2025-10-31 03:41:09
AS:159.65.255.114                                         -
------------------------------------------------------------------------------
```

## OpenVPN Linux cli client openvpn3

<https://openvpn.net/community-docs/openvpn-client-for-linux.html>

```console
ubuntu@LAPTOP-JBell:~$ openvpn3 session-start --config AS:159.65.255.114
Using pre-loaded configuration profile 'AS:159.65.255.114'
Session path: /net/openvpn/v3/sessions/debf947csf0cds48bbsa019s946d6bc46a65
Auth User name: openvpn
Auth Password:
Connected to 159.65.255.114 (159.65.255.114)
```

```console
ubuntu@LAPTOP-JBell:~$ openvpn3 sessions-list
-----------------------------------------------------------------------------
        Path: /net/openvpn/v3/sessions/debf947csf0cds48bbsa019s946d6bc46a65
     Created: 2025-11-02 14:14:28                       PID: 188942
       Owner: ubuntu                                 Device: tun0
 Config name: AS:159.65.255.114
Connected to: udp:159.65.255.114:1194
      Status: Connection, Client connected
-----------------------------------------------------------------------------
```

We check the public IP of L using <https://www.my-ip.io/> which has nice result formatting endpoints.

```console
ubuntu@LAPTOP-JBell:~$ curl https://api.my-ip.io/v2/ip.txt
159.65.255.114
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
159.65.0.0/16
```

We use <https://proof.ovh.net/files/>:

```console
ubuntu@LAPTOP-JBell:~$ curl -L -O https://proof.ovh.net/files/100Mb.dat
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  100M  100  100M    0     0  3257k      0  0:00:31  0:00:31 --:--:-- 6520k
```

```console
ubuntu@LAPTOP-JBell:~$ openvpn3 session-manage --config AS:159.65.255.114 --disconnect
Initiated session shutdown.

Connection statistics:
     BYTES_IN...............110958336
     BYTES_OUT................1164111
     PACKETS_IN.................77478
     PACKETS_OUT................14853
     TUN_BYTES_IN..............803153
     TUN_BYTES_OUT..........109094754
     TUN_PACKETS_IN.............14762
     TUN_PACKETS_OUT............77386
```

```console
ubuntu@LAPTOP-JBell:~$ echo "100 * 1024 * 1024" | bc
104857600
```

## OpenVPN Connect for Windows

Download and install OpenVPN Connect for Windows: <https://openvpn.net/client/>

Download Connection Profile from OpenVPN Access Server web server <https://159.65.255.114/>: `profile-userlocked.ovpn`

Upload configuration file `profile-userlocked.ovpn` to Windows client.

We download <https://proof.ovh.net/files/100Mb.dat> and observe the OpenVPN connection statistics.

--

<img width="250" height="428" alt="image" src="https://github.com/user-attachments/assets/d36d7225-e14e-4a54-b2bc-7f980101c71e" />

--

<img width="500" height="857" alt="image" src="https://github.com/user-attachments/assets/2fcbee68-1658-4900-9308-9e4e54363c1c" />
