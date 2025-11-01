# OpenVPN Part 2

 and `KEY_NAME=EASY_RSA`

## Connect from L to S as openvpn user

From machine L, we connect to machine S using SSH:

```console
ubuntu@LAPTOP-JBell:~$ ssh -i .ssh/OPEN_VPN -l openvpn 159.203.164.175
```

As user openvpn in machine S,

```console
openvpn@openvpnaccessserver2143onubuntu2404-s-1vcpu-2gb-70gb-intel-nyc3:~$ curl -L -o EasyRSA-3.2.4.tgz https://github.com/OpenVPN/easy-rsa/releases/download/v3.2.4/EasyRSA-3.2.4.tgz
openvpn@openvpnaccessserver2143onubuntu2404-s-1vcpu-2gb-70gb-intel-nyc3:~$ KEYWORD=unknown-keyword
openvpn@openvpnaccessserver2143onubuntu2404-s-1vcpu-2gb-70gb-intel-nyc3:~$ tar -xzf EasyRSA-3.2.4.tgz --warning=no-$KEYWORD
openvpn@openvpnaccessserver2143onubuntu2404-s-1vcpu-2gb-70gb-intel-nyc3:~$ cd EasyRSA-3.2.4/
easyrsa@EASY-RSA-s-1vcpu-1gb-35gb-intel-sfo2-01:~/EasyRSA-3.2.4$ ./easyrsa init-pki

Notice
------
'init-pki' complete; you may now create a CA or requests.

Your newly created PKI dir is:
* /home/easyrsa/EasyRSA-3.2.4/pki

Using Easy-RSA configuration:
* undefined
easyrsa@EASY-RSA-s-1vcpu-1gb-35gb-intel-sfo2-01:~/EasyRSA-3.2.4$ ./easyrsa build-ca

Enter New CA Key Passphrase:

Confirm New CA Key Passphrase:
......+.....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.......+..+....+.....+......+......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*...+....+...+..+......................+......+..+......+.+........+.+............+..+.+.....+............+...+....+........+..........+.................+...+....+...........+.........+....+......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
...+..........+.........+.........+.....+.+.....+.........+......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+...+.........+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*........+...+.+...........+...+......+.+........+...............................+...........+......+.......+...........+.+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
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


## Connect from L to S using SSH private key

We use the private key on local machine L to connect as root to remote machine R.[^connect-droplet]

[^connect-droplet]: <https://docs.digitalocean.com/products/droplets/how-to/connect-with-ssh/openssh/>

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

T

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

## Make droplet for server R (easy-rsa server)

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
easyrsa@EASY-RSA-s-1vcpu-1gb-35gb-intel-sfo2-01:~/EasyRSA-3.2.4$ ./easyrsa init-pki

Notice
------
'init-pki' complete; you may now create a CA or requests.

Your newly created PKI dir is:
* /home/easyrsa/EasyRSA-3.2.4/pki

Using Easy-RSA configuration:
* undefined
easyrsa@EASY-RSA-s-1vcpu-1gb-35gb-intel-sfo2-01:~/EasyRSA-3.2.4$ ./easyrsa build-ca

Enter New CA Key Passphrase:

Confirm New CA Key Passphrase:
...+.........+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+.......+..................+......+..+...+.+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.....+.........+....+...+...+..+.............+..+....+...+...+..+.....................+.+...............+.....+......+...+....+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
.....+......+..+.............+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*......+...+.....+....+..+.+...............+............+.....+...+.......+..................+.....+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
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
