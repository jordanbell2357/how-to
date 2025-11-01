# ssh

L = local Linux machine (WSL)
R = remote Linux machine (DigitalOcean droplet)
S = other remote Linux machine (DigitalOcean droplet)
C = Cloudflare

> SSH provides an authentication mechanism based on cryptographic keys, called public key authentication.
> One or more public keys may be configured as authorized keys; the private key corresponding to an authorized key serves as authentication to the server.
> Typically both authorized keys and private keys are stored in the .ssh directory in a user's home directory.
> Fundamentally, such keys are like fancy passwords, only the password cannot be stolen from the network and it is possible to encrypt the private key locally
> (so that using it requires both a file and a passphrase only known to a user). However, in practice most keys are used for automation and do not have a passphrase. [^ssh]

[^ssh]: <https://www.ssh.com/academy/ssh/protocol>

We make two SSH key pairs, using the following with `KEY_NAME=EASY_RSA` and then `KEY_NAME=OPEN_VPN`. [^ssh-key-gen] (For "production", we'd want differnent SSH keys associated with each user
on each remote machine, so we'd make four instead of two.)

[^ssh-key-gen]: <https://www.ssh.com/academy/ssh/keygen>

```console
ubuntu@LAPTOP-JBell:~$ KEY_NAME=EASY_RSA
ubuntu@LAPTOP-JBell:~$ ssh-keygen -t ed25519 -C "$KEY_NAME" -f ~/.ssh/$KEY_NAME
Generating public/private ed25519 key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ubuntu/.ssh/EASY_RSA
Your public key has been saved in /home/ubuntu/.ssh/EASY_RSA.pub
The key fingerprint is:
SHA256:+vumAe+B3nBkoOscuQMwmBPHoCIkMPB4Z9BhsZ4sKfc EASY_RSA
The key's randomart image is:
+--[ED25519 256]--+
|O+..+o           |
|=+oo..           |
|*+o + .          |
|O. * o .         |
|.++ = . S        |
| o.o o B         |
|   .E + =        |
|   o.+ * o.      |
|    +.. *=.      |
+----[SHA256]-----+
ubuntu@LAPTOP-JBell:~$ cat .ssh/$KEY_NAME.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOKivAF1DIVMFYBT0wK+kCPYmzO3YuAxWadpHG/Bws1f EASY_RSA
```

We add the public keys to DigitalOcean, make droplets R and S, and associate one public key with each droplet.

We use the private key on local machine L to connect as root to remote machine R.

```console
ubuntu@LAPTOP-JBell:~$ ssh -i .ssh/$KEY_NAME -l root 206.189.160.22
The authenticity of host '206.189.160.22 (206.189.160.22)' can't be established.
ED25519 key fingerprint is SHA256:icViKvZ3NxfP61wakkToByDXABM2Oigm2nXPIdlvMi4.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '206.189.160.22' (ED25519) to the list of known hosts.
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.8.0-71-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Sat Nov  1 02:05:07 UTC 2025

  System load:           0.0
  Usage of /:            5.7% of 32.86GB
  Memory usage:          19%
  Swap usage:            0%
  Processes:             101
  Users logged in:       0
  IPv4 address for eth0: 206.189.160.22
  IPv4 address for eth0: 10.46.0.5
  IPv6 address for eth0: 2604:a880:2:d1::e6bb:2001

Expanded Security Maintenance for Applications is not enabled.

72 updates can be applied immediately.
39 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.
```

The public key of EASY_RSA is installed in the `authorized_keys` file on machine R: [^authorized-keys]

[^authorized-keys]: <https://www.ssh.com/academy/ssh/authorized-keys-openssh>

```console
root@EASY-RSA-s-1vcpu-1gb-35gb-intel-sfo2-01:/# cat /etc/ssh/sshd_config | grep AuthorizedKeysFile
#AuthorizedKeysFile     .ssh/authorized_keys .ssh/authorized_keys2
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOKivAF1DIVMFYBT0wK+kCPYmzO3YuAxWadpHG/Bws1f EASY_RSA
root@EASY-RSA-s-1vcpu-1gb-35gb-intel-sfo2-01:~# cat .ssh/authorized_keys2
cat: .ssh/authorized_keys2: No such file or directory
```

On machine R, we make a user. [^adduser]

[^adduser]: https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu

```console
root@EASY-RSA-s-1vcpu-1gb-35gb-intel-sfo2-01:~# adduser easyrsa
info: Adding user `easyrsa' ...
info: Selecting UID/GID from range 1000 to 59999 ...
info: Adding new group `easyrsa' (1000) ...
info: Adding new user `easyrsa' (1000) with group `easyrsa (1000)' ...
info: Creating home directory `/home/easyrsa' ...
info: Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for easyrsa
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] Y
info: Adding new user `easyrsa' to supplemental / extra groups `users' ...
info: Adding user `easyrsa' to group `users' ...
```

```console
root@EASY-RSA-s-1vcpu-1gb-35gb-intel-sfo2-01:~# usermod -aG sudo easyrsa
```

Set up ufw:

```console
root@EASY-RSA-s-1vcpu-1gb-35gb-intel-sfo2-01:~# ufw app list
Available applications:
  OpenSSH
root@EASY-RSA-s-1vcpu-1gb-35gb-intel-sfo2-01:~# ufw allow OpenSSH
Rules updated
Rules updated (v6)
root@EASY-RSA-s-1vcpu-1gb-35gb-intel-sfo2-01:~# ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
root@EASY-RSA-s-1vcpu-1gb-35gb-intel-sfo2-01:~# ufw status
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
```

Then use rsync to copy the SSH key pair from the root user home folder to the easyrsa user home folder: [^rsync]

[^rsync]: <https://download.samba.org/pub/rsync/rsync.1>

```console
root@EASY-RSA-s-1vcpu-1gb-35gb-intel-sfo2-01:~# rsync --archive --chown=easyrsa:easyrsa ~/.ssh /home/easyrsa
root@EASY-RSA-s-1vcpu-1gb-35gb-intel-sfo2-01:~# ls -l /home/easyrsa/.ssh/
total 4
-rw------- 1 easyrsa easyrsa 90 Nov  1 02:00 authorized_keys
root@EASY-RSA-s-1vcpu-1gb-35gb-intel-sfo2-01:~# cat /home/easyrsa/.ssh/authorized_keys
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOKivAF1DIVMFYBT0wK+kCPYmzO3YuAxWadpHG/Bws1f EASY_RSA
```

Now on machine L, we use SSH to connect as user easyrsa to machine R.

```console
ubuntu@LAPTOP-JBell:~$ ssh -i .ssh/EASY_RSA -l easyrsa 206.189.160.22
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.8.0-71-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Sat Nov  1 03:00:02 UTC 2025

  System load:           0.0
  Usage of /:            5.7% of 32.86GB
  Memory usage:          19%
  Swap usage:            0%
  Processes:             104
  Users logged in:       1
  IPv4 address for eth0: 206.189.160.22
  IPv4 address for eth0: 10.46.0.5
  IPv6 address for eth0: 2604:a880:2:d1::e6bb:2001

Expanded Security Maintenance for Applications is not enabled.

72 updates can be applied immediately.
39 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.
```

Now we install easy-rsa. [^easy-rsa]

[^easy-rsa]: <https://github.com/OpenVPN/easy-rsa/releases/tag/v3.2.4>

```console
easyrsa@EASY-RSA-s-1vcpu-1gb-35gb-intel-sfo2-01:~$ curl -L -o EasyRSA-3.2.4.tgz https://github.com/OpenVPN/easy-rsa/releases/download/v3.2.4/EasyRSA-3.2.4.tgz
easyrsa@EASY-RSA-s-1vcpu-1gb-35gb-intel-sfo2-01:~$ tar -xzf EasyRSA-3.2.4.tgz
tar: Ignoring unknown extended header keyword 'LIBARCHIVE.xattr.com.apple.metadata:kMDItemTextContentLanguage'
```

This warning occurs because the tgz file was made using bsdtar from libarchive [^libarchive] rather than with GNU tar.

[^libarchive]: `sudo apt install libarchive-tools` to install bsdtar and then `bsdtar -xzf EasyRSA-3.2.4.tgz`.

We disable warnings in GNU tar with the option <tt>--warning=no-$KEYWORD</tt> where KEYWORD, and here we use KEYWORD=unknown-keyword [^gnutar]

[^gnutar]: <https://www.gnu.org/software/tar/manual/html_node/warnings.html> and <https://www.gnu.org/software/tar/manual/html_node/Archive-Extraction-Warnings.html>

```console
easyrsa@EASY-RSA-s-1vcpu-1gb-35gb-intel-sfo2-01:~$ KEYWORD=unknown-keyword
easyrsa@EASY-RSA-s-1vcpu-1gb-35gb-intel-sfo2-01:~$ tar -xzf EasyRSA-3.2.4.tgz --warning=no-$KEYWORD
```

Then use easy-rsa: [^easy-rsa-docs]

[^easy-rsa-docs]: <https://community.openvpn.net/Pages/EasyRSA3-OpenVPN-Howto>

```console
easyrsa@EASY-RSA-s-1vcpu-1gb-35gb-intel-sfo2-01:~/EasyRSA-3.2.4$ ./easyrsa build-ca nopass
........+........+.......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*......+.....+...+....+........+............+...............+.+..+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*..+.......+..............+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
.....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.............+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+...+...+..+...+.........+..................+....+......+..+.+.........+..+.+..+............+....+.........+........+.........+................+..+...+.......+..+.........+.......+........+.+...+...+...+..............+......+......+......+...+............+.............+..+....+...+......+..+..........+..+...+....+...+..+.........+.+.........+..+....+...+...........+.+...+..+.+..................+..+..................+....+.....+.+..........................+.+..+...............+............+......+...+......+...+..........+......+..+...+.+...............+.....+....+.....+............+.......+..+..........+...+...+...+.................+....+...+.....+......+...............+.+........+.+.....+.+........+......+.........+...+....+.....+......+.......+..+.+.........+.........+............+........+....+......+......+.....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:

Notice
------
CA creation complete. Your new CA certificate is at:
* /home/easyrsa/EasyRSA-3.2.4/pki/ca.crt

Build-ca completed successfully.
```

Now we use SSH to connect from L to system S. First we activate the OpenVPN Access Server.

```console
ubuntu@LAPTOP-JBell:~$ ssh -i .ssh/OPEN_VPN -l root 159.203.164.175
The authenticity of host '159.203.164.175 (159.203.164.175)' can't be established.
ED25519 key fingerprint is SHA256:RopZkdq6IqhhticLRdj41aVtPgsu/uwig5NyO1iYny8.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '159.203.164.175' (ED25519) to the list of known hosts.
Welcome to OpenVPN Access Server Appliance 2.14.3

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Sat Nov  1 05:57:18 UTC 2025

  System load:           0.0
  Usage of /:            4.1% of 66.76GB
  Memory usage:          14%
  Swap usage:            0%
  Processes:             100
  Users logged in:       0
  IPv4 address for eth0: 159.203.164.175
  IPv4 address for eth0: 10.17.0.5
  IPv6 address for eth0: 2604:a880:800:14:0:1:ee88:4000

Expanded Security Maintenance for Applications is not enabled.

29 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


*** System restart required ***

          OpenVPN Access Server
          Initial Configuration Tool
```


```console
Please enter 'yes' to indicate your agreement [no]: yes
Once you provide a few initial configuration settings,
OpenVPN Access Server can be configured by accessing
its Admin Web UI using your Web browser.

Will this be the primary Access Server node?
(enter 'no' to configure as a backup or standby node)
> Press ENTER for default [yes]:yes

Please specify the network interface and IP address to be
used by the Admin Web UI:
(1) all interfaces: 0.0.0.0
(2) eth0: 159.203.164.175
(3) eth0: 10.17.0.5
(4) eth1: 10.108.0.3
Please enter the option number from the list above (1- 4).
> Press Enter for default [1]:

What public/private type/algorithms do you want to use for the OpenVPN CA?

Recommended choices:

rsa       - maximum compatibility
secp384r1 - elliptic curve, higher security than rsa, allows faster connection setup and smaller user profile files
showall   - shows all options including non-recommended algorithms.
> Press ENTER for default [secp384r1]:

Please specify the port number for the Admin Web UI.
> Press ENTER for default [943]:

Please specify the TCP port number for the OpenVPN Daemon
> Press ENTER for default [443]:

Should client traffic be routed by default through the VPN?
> Press ENTER for default [yes]:

Should client DNS traffic be routed by default through the VPN?
> Press ENTER for default [yes]:
Admin user authentication will be  local

Private subnets detected: ['10.17.0.0/20', '10.108.0.0/20']

Should private subnets be accessible to clients by default?
> Press ENTER for default [yes]:

To initially login to the Admin Web UI, you must use a
username and password that successfully authenticates you
with the host UNIX system (you can later modify the settings
so that RADIUS or LDAP is used for authentication instead).

You can login to the Admin Web UI as "openvpn" or specify
a different user account to use for this purpose.

Do you wish to login to the Admin UI as "openvpn"?
> Press ENTER for default [yes]:
Type a password for the 'openvpn' account (if left blank, a random password will be generated):
Error: Password must contain an Uppercase letter, and a symbol from !@#$%&'()*+,-/[\]^_`{|}~<>
Type a password for the 'openvpn' account (if left blank, a random password will be generated):
Confirm the password for the 'openvpn' account:
```

Input activation key for OpenVPN, from <https://as-portal.openvpn.com/instructions/digital-ocean/activation>

```console
Activation succeeded



Initializing OpenVPN...
Removing Cluster Admin user login...
userdel: user 'admin_c' does not exist
Writing as configuration file...
Perform sa init...
Wiping any previous userdb...
Creating default profile...
Modifying default profile...
Adding new user to userdb...
Modifying new user as superuser in userdb...
Setting password in db...
Getting hostname...
Hostname: ASBuildImage-ubuntu24-v2
Preparing web certificates...
Getting web user account...
Adding web group account...
Adding web group...
groupadd: group 'openvpn_as' already exists
Adjusting license directory ownership...
chown: warning: '.' should be ':': ‘openvpn_as.openvpn_as’
Initializing confdb...
Initial version is not set. Setting it to 2.14.3...
Generating PAM config for openvpnas ...
Enabling service
Created symlink /etc/systemd/system/multi-user.target.wants/openvpnas.service → /usr/lib/systemd/system/openvpnas.service.
Starting openvpnas...

NOTE: Your system clock must be correct for OpenVPN Access Server
to perform correctly.  Please ensure that your time and date
are correct on this system.

Initial Configuration Complete!

You can now continue configuring OpenVPN Access Server by
directing your Web browser to this URL:

https://159.203.164.175:943/admin

During normal operation, OpenVPN AS can be accessed via these URLs:
Admin  UI: https://159.203.164.175:943/admin
Client UI: https://159.203.164.175:943/
To login please use the "openvpn" account with the password you specified during the setup.

See the Release Notes for this release at:
   https://openvpn.net/vpn-server-resources/release-notes/
```

We now make a non-root user on machine S.

```console
root@ASBuildImage-ubuntu24-v2:~# adduser openvpn
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
root@ASBuildImage-ubuntu24-v2:~# usermod -aG sudo openvpn
```

Then

```console
root@ASBuildImage-ubuntu24-v2:~# ufw app list
Available applications:
  OpenSSH
root@ASBuildImage-ubuntu24-v2:~# ufw allow OpenSSH
Rules updated
Rules updated (v6)
root@ASBuildImage-ubuntu24-v2:~# ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
root@ASBuildImage-ubuntu24-v2:~# ufw status
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
```

```console
root@ASBuildImage-ubuntu24-v2:~# reboot
root@ASBuildImage-ubuntu24-v2:~# Connection to 159.203.164.175 closed by remote host.
Connection to 159.203.164.175 closed.
```

Using rsync with machine S

```console
root@openvpnaccessserver2143onubuntu2404-s-1vcpu-2gb-70gb-intel-nyc3:~# rsync --archive --chown=openvpn:openvpn ~/.ssh /home/openvpn
```

Then from machine L, we connect to machine S using SSH:

```console
ubuntu@LAPTOP-JBell:~$ ssh -i .ssh/OPEN_VPN -l openvpn 159.203.164.175
Welcome to OpenVPN Access Server Appliance 2.14.3

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Sat Nov  1 06:38:24 UTC 2025

  System load:           0.0
  Usage of /:            4.1% of 66.76GB
  Memory usage:          22%
  Swap usage:            0%
  Processes:             114
  Users logged in:       1
  IPv4 address for eth0: 159.203.164.175
  IPv4 address for eth0: 10.17.0.5
  IPv6 address for eth0: 2604:a880:800:14:0:1:ee88:4000

Expanded Security Maintenance for Applications is not enabled.

29 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.
```

Now as user openvpn in machine S we do

```console
openvpn@openvpnaccessserver2143onubuntu2404-s-1vcpu-2gb-70gb-intel-nyc3:~$ KEYWORD=unknown-keyword
openvpn@openvpnaccessserver2143onubuntu2404-s-1vcpu-2gb-70gb-intel-nyc3:~$ tar -xzf EasyRSA-3.2.4.tgz --warning=no-$KEYWORD
openvpn@openvpnaccessserver2143onubuntu2404-s-1vcpu-2gb-70gb-intel-nyc3:~$ cd EasyRSA-3.2.4/
openvpn@openvpnaccessserver2143onubuntu2404-s-1vcpu-2gb-70gb-intel-nyc3:~/EasyRSA-3.2.4$ ./easyrsa init-pki

Notice
------
'init-pki' complete; you may now create a CA or requests.

Your newly created PKI dir is:
* /home/openvpn/EasyRSA-3.2.4/pki

Using Easy-RSA configuration:
* undefined
```
