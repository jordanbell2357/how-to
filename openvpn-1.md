# OpenVPN Part 1

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

We make an SSH key pairs using `KEY_NAME=OPEN_VPN`. [^ssh-key-gen]

[^ssh-key-gen]: <https://www.ssh.com/academy/ssh/keygen>

```console
ubuntu@LAPTOP-JBell:~$ KEY_NAME=OPEN_VPN
ubuntu@LAPTOP-JBell:~$ ssh-keygen -t ed25519 -C "$KEY_NAME" -f ~/.ssh/$KEY_NAME
Generating public/private ed25519 key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ubuntu/.ssh/OPEN_VPN
Your public key has been saved in /home/ubuntu/.ssh/OPEN_VPN.pub

ubuntu@LAPTOP-JBell:~$ cat .ssh/$KEY_NAME.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFwJ1Lw8LwnsmZZRd0AQ5arvqfNqZ0Y59wm9vtdozZiH OPEN_VPN
```

## Create DigitalOcean account

<https://www.digitalocean.com/>

## Add SSH public key <tt>OPEN_VPN.pub</tt> to DigitalOcean account

We then add the public key <tt>OPEN_VPN.pub</tt> to DigitalOcean and will associate the key with a droplet, following
<https://docs.digitalocean.com/platform/teams/how-to/upload-ssh-keys/>

## Create OpenVPN account

<https://openvpn.net/access-server/>

## Deploy Access Server S

Associate SSH public key <tt>OPEN_VPN.pub</tt> to droplet.

<https://as-portal.openvpn.com/instructions/digital-ocean/installation>

<https://openvpn.net/as-docs/digitalocean.html>

## Connect from L to S as root user using SSH private key OPEN_VPN and activate

Now we use SSH to connect from L to system S. First we activate the OpenVPN Access Server S by accessing it with SSH, and using the
activation key in <https://as-portal.openvpn.com/instructions/digital-ocean/activation>

We make a user <tt>openvpn</tt>.

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
<<https://as-portal.openvpn.com/instructions/digital-ocean/activation>>

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

Finally

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

## Make non-root user on OpenVPN Access Server S and give private key OPEN_VPN

We now make a non-root user on machine S.

```console
root@ASBuildImage-ubuntu24-v2:~# adduser openvpn
```

```console
root@ASBuildImage-ubuntu24-v2:~# usermod -aG sudo openvpn
```

We have been able to access S from L because the public key of OPEN_VPN is installed on droplet creation in the `authorized_keys` file on machine S: [^authorized-keys]

[^authorized-keys]: <https://www.ssh.com/academy/ssh/authorized-keys-openssh>

```console
root@ASBuildImage-ubuntu24-v2:~# cat /etc/ssh/sshd_config | grep AuthorizedKeysFile
#AuthorizedKeysFile     .ssh/authorized_keys .ssh/authorized_keys2
root@ASBuildImage-ubuntu24-v2:~# cat .ssh/authorized_keys
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFwJ1Lw8LwnsmZZRd0AQ5arvqfNqZ0Y59wm9vtdozZiH OPEN_VPN
```

We add the OPEN_VPN public key to the `authorized_keys` of user <tt>openvpn</tt>

Then use rsync to copy the SSH key pair from the root user home folder of S to the openvpn user home folder of S: [^rsync]

[^rsync]: <https://download.samba.org/pub/rsync/rsync.1>

```console
root@EASY-RSA-s-1vcpu-1gb-35gb-intel-sfo2-01:~# rsync --archive --chown=openvpn:openvpn ~/.ssh /home/openvpn
root@ASBuildImage-ubuntu24-v2:~# cat /home/openvpn/.ssh/authorized_keys
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFwJ1Lw8LwnsmZZRd0AQ5arvqfNqZ0Y59wm9vtdozZiH OPEN_VPN
```
