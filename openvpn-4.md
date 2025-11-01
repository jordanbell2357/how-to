



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





# Server T

```console
ubuntu@LAPTOP-JBell:~$ ssh-keygen -t ed25519 -C "SERVER" -f ~/.ssh/SERVER
```

ubuntu@LAPTOP-JBell:~$ cat .ssh/SERVER.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINZM5N1qU8nnOJ+YwI0Re5gE6QunSUoig0sGX7jJ/Pgu SERVER


root@server-s-1vcpu-1gb-35gb-intel-nyc3-01:~# dpkg-reconfigure tzdata

Current default time zone: 'America/New_York'
Local time is now:      Sat Nov  1 14:11:15 EDT 2025.
Universal Time is now:  Sat Nov  1 18:11:15 UTC 2025.

root@server-s-1vcpu-1gb-35gb-intel-nyc3-01:~# adduser server

root@server-s-1vcpu-1gb-35gb-intel-nyc3-01:~# usermod -aG sudo server

root@server-s-1vcpu-1gb-35gb-intel-nyc3-01:~# rsync --archive --chown=server:server ~/.ssh /home/server

root@server-s-1vcpu-1gb-35gb-intel-nyc3-01:~# cat /home/server/.ssh/authorized_keys
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINZM5N1qU8nnOJ+YwI0Re5gE6QunSUoig0sGX7jJ/Pgu SERVER
