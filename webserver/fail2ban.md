# fail2ban

<https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-20-04>

```console
ubuntu@vps-9e6a8f0e:~$ sudo apt install fail2ban
```

```console
ubuntu@vps-9e6a8f0e:~$ sudo systemctl enable fail2ban
Synchronizing state of fail2ban.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable fail2ban
```

we look at sshd logs.

```console
ubuntu@vps-9e6a8f0e:/var/www/jordanbell.org/public_html$ journalctl -u ssh.service | grep -i invalid | wc -l
2427
```

```console
ubuntu@vps-9e6a8f0e:/var/www/jordanbell.org/public_html$ journalctl -u ssh.service | shuf -n 10
Nov 17 03:56:57 vps-9e6a8f0e sshd[12078]: Connection closed by 109.236.83.55 port 54920 [preauth]
Nov 17 00:54:52 vps-9e6a8f0e sshd[9827]: Received disconnect from 41.216.181.70 port 57416:11: Bye Bye [preauth]
Nov 17 03:51:56 vps-9e6a8f0e sshd[12004]: Connection closed by authenticating user ubuntu 143.244.143.91 port 47674 [preauth]
Nov 17 03:06:24 vps-9e6a8f0e sshd[11407]: Connection closed by authenticating user root 193.32.162.157 port 33386 [preauth]
Nov 17 01:57:21 vps-9e6a8f0e sshd[10594]: Invalid user admin from 103.49.62.60 port 38990
Nov 17 01:49:38 vps-9e6a8f0e sshd[10376]: Connection closed by authenticating user root 68.183.87.87 port 43222 [preauth]
Nov 17 00:58:11 vps-9e6a8f0e sshd[9854]: Disconnected from invalid user kguzman 136.228.161.66 port 37656 [preauth]
Nov 17 02:54:51 vps-9e6a8f0e sshd[11180]: Invalid user ahmed from 193.32.162.157 port 54384
Nov 17 01:56:16 vps-9e6a8f0e sshd[10498]: Connection closed by invalid user guestuser 103.49.62.60 port 60800 [preauth]
Nov 16 21:08:12 vps-9e6a8f0e sshd[3612]: Invalid user admin from 134.199.152.94 port 39044
```

Repeating this a handful of times, we make the following list of sshd log pattern types. We don't distinguish between root and any other username for classifying login attempts.

```
Nov 17 03:56:57 vps-9e6a8f0e sshd[12078]: Connection closed by 109.236.83.55 port 54920 [preauth]
Nov 17 01:56:16 vps-9e6a8f0e sshd[10498]: Connection closed by invalid user guestuser 103.49.62.60 port 60800 [preauth]
Nov 17 03:51:56 vps-9e6a8f0e sshd[12004]: Connection closed by authenticating user ubuntu 143.244.143.91 port 47674 [preauth]
Nov 17 00:31:48 vps-9e6a8f0e sshd[9603]: Disconnected from authenticating user root 160.22.123.78 port 51950 [preauth]
Nov 17 00:58:11 vps-9e6a8f0e sshd[9854]: Disconnected from invalid user kguzman 136.228.161.66 port 37656 [preauth]
Nov 17 01:57:21 vps-9e6a8f0e sshd[10594]: Invalid user admin from 103.49.62.60 port 38990
Nov 17 00:54:52 vps-9e6a8f0e sshd[9827]: Received disconnect from 41.216.181.70 port 57416:11: Bye Bye [preauth]
```

We extract incoming IP addresses:

```bash
journalctl -u ssh.service | grep -e "Connection closed by invalid user" | sed -r "s/.*by invalid user [[:graph:]]+? (.*) port [0-9]+.*/\1/"
```

```bash
journalctl -u ssh.service |
grep -e "Connection closed by invalid user" |
sed -r "s/.*by invalid user [[:graph:]]+? (.*) port [0-9]+.*/\1/" |
sort |
uniq
```

```
101.36.123.102
101.43.48.241
103.49.62.60
104.248.10.213
104.248.85.23
113.132.113.56
114.111.54.188
116.198.207.211
116.99.170.40
119.148.37.233
129.212.177.237
129.212.179.35
134.199.152.94
134.199.155.220
134.199.202.138
139.59.68.64
143.244.143.91
146.190.16.121
149.104.26.75
159.89.163.141
161.35.90.195
167.172.43.167
170.64.233.194
171.231.195.194
176.65.148.161
193.32.162.157
194.87.252.171
195.178.110.30
2.57.122.177
209.38.42.79
27.79.0.42
27.79.2.74
27.79.43.185
27.79.5.142
43.156.233.89
45.148.10.240
47.238.66.99
47.96.1.152
5.129.218.123
61.245.11.87
64.226.82.6
65.49.1.173
68.183.13.195
68.183.87.87
77.90.185.47
8.153.202.82
8.219.56.235
80.94.92.40
92.118.39.62
92.118.39.87
92.118.39.92
92.118.39.95
```


```console
ubuntu@vps-9e6a8f0e:/etc/fail2ban$ sudo fail2ban-client status sshd
Status for the jail: sshd
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     0
|  `- Journal matches:  _SYSTEMD_UNIT=sshd.service + _COMM=sshd
`- Actions
   |- Currently banned: 4
   |- Total banned:     4
   `- Banned IP list:   118.193.61.63 159.223.37.230 64.227.160.144 96.77.54.153
```
