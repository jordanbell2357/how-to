# journalctl

<https://www.loggly.com/ultimate-guide/troubleshooting-with-linux-logs/>

<https://www.loggly.com/ultimate-guide/using-journalctl/>

<https://www.freedesktop.org/software/systemd/man/latest/journalctl.html>

## ssh.service

After five or fewer days of operating a server, but revealing the user name, IP address and domain name in GitHub,
there are the following SSH login attempts attempted:

```console
ubuntu@vps-9e6a8f0e:~$ journalctl -u ssh.service | grep "Failed password" | wc -l
13324
```

```console
ubuntu@vps-9e6a8f0e:~$ TZ="America/New_York" date; journalctl -u ssh.service | grep "Failed password" | wc -l
Thu Nov  6 17:52:32 EST 2025
26775
```

```console
ubuntu@vps-9e6a8f0e:~$ TZ="America/New_York" date; journalctl -u ssh.service | grep -v "Failed password" | wc -l
Thu Nov  6 17:53:30 EST 2025
73194
```

```console
ubuntu@vps-9e6a8f0e:~$ TZ="America/New_York" date; cat /var/log/auth.log | wc -l
Thu Nov  6 17:54:50 EST 2025
100815
```

```console
ubuntu@vps-9e6a8f0e:~$ TZ="America/New_York" date; journalctl -n 10 --no-pager -u ssh.service
Sat Nov  8 09:14:51 EST 2025
Nov 08 14:14:00 vps-9e6a8f0e sshd[96916]: Failed password for invalid user admin from 2.57.121.112 port 54765 ssh2
Nov 08 14:14:00 vps-9e6a8f0e sshd[96916]: pam_unix(sshd:auth): check pass; user unknown
Nov 08 14:14:02 vps-9e6a8f0e sshd[96916]: Failed password for invalid user admin from 2.57.121.112 port 54765 ssh2
Nov 08 14:14:03 vps-9e6a8f0e sshd[96916]: Received disconnect from 2.57.121.112 port 54765:11: Bye [preauth]
Nov 08 14:14:03 vps-9e6a8f0e sshd[96916]: Disconnected from invalid user admin 2.57.121.112 port 54765 [preauth]
Nov 08 14:14:03 vps-9e6a8f0e sshd[96916]: PAM 4 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=2.57.121.112
Nov 08 14:14:03 vps-9e6a8f0e sshd[96916]: PAM service(sshd) ignoring max retries; 5 > 3
Nov 08 14:14:23 vps-9e6a8f0e sshd[96918]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=23.248.211.58  user=root
Nov 08 14:14:25 vps-9e6a8f0e sshd[96918]: Failed password for root from 23.248.211.58 port 46190 ssh2
Nov 08 14:14:26 vps-9e6a8f0e sshd[96918]: Connection closed by authenticating user root 23.248.211.58 port 46190 [preauth]
```


## --disk-usage and --vacuum-size

```console
ubuntu@vps-9e6a8f0e:~$ journalctl --disk-usage
Archived and active journals take up 111.3M in the file system.
```


```console
ubuntu@vps-9e6a8f0e:~$ sudo journalctl --vacuum-size=100M
Deleted archived journal /var/log/journal/412bb863510e4a86931db99b7219d0a3/system@c010fc4efbda48be93a58a9d76038120-000000000000026d-000642aa15a0d871.journal (12.5M).
Deleted archived journal /var/log/journal/412bb863510e4a86931db99b7219d0a3/user-1000@c010fc4efbda48be93a58a9d76038120-00000000000006cb-000642aa286e5101.journal (1.1M).
Deleted archived journal /var/log/journal/412bb863510e4a86931db99b7219d0a3/system@c010fc4efbda48be93a58a9d76038120-00000000000048a3-000642b5a3d960a6.journal (7.1M).
Vacuuming done, freed 20.8M of archived journals from /var/log/journal/412bb863510e4a86931db99b7219d0a3.
Vacuuming done, freed 0B of archived journals from /run/log/journal.
Vacuuming done, freed 0B of archived journals from /var/log/journal.
```

```console
ubuntu@vps-9e6a8f0e:~$ journalctl --disk-usage
Archived and active journals take up 98.5M in the file system.
```

```console
ubuntu@vps-9e6a8f0e:~$ TZ="America/New_York" date; journalctl -u ssh.service | grep -v "Failed password" | wc -l
Thu Nov  6 17:56:05 EST 2025
58376
ubuntu@vps-9e6a8f0e:~$ TZ="America/New_York" date; journalctl -u ssh.service | grep -v "Failed password" | wc -l
Thu Nov  6 17:56:20 EST 2025
58387
```

## OOM

```console
ubuntu@vps-9e6a8f0e:~$ vmstat -S M
procs -----------memory---------- ---swap-- -----io---- -system-- -------cpu-------
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st gu
 1  0      0   7500      4     62    0    0     2    19   55    0  0  0 100  0  0
```

There are 8G of memroy. Thus we trigger the OOM killer with the following:


```console
ubuntu@vps-9e6a8f0e:~$ head -c 10G /dev/zero | tail > /dev/null; echo $?
Killed
137
```

Checking logs with dmesg:

```console
ubuntu@vps-9e6a8f0e:~$ sudo dmesg | tail -n 2
[253894.889279] oom-kill:constraint=CONSTRAINT_NONE,nodemask=(null),cpuset=/,mems_allowed=0,global_oom,task_memcg=/user.slice/user-1000.slice/session-511.scope,task=tail,pid=51714,uid=1000
[253894.889301] Out of memory: Killed process 51714 (tail) total-vm:7627620kB, anon-rss:7621248kB, file-rss:1408kB, shmem-rss:0kB, UID:1000 pgtables:14968kB oom_score_adj:0
```

Checking logs with `cat /var/log/syslog`:

```console
ubuntu@vps-9e6a8f0e:~$ cat /var/log/syslog | grep -i "Out of memory"
2025-11-06T23:06:45.530571+00:00 vps-9e6a8f0e kernel: Out of memory: Killed process 51714 (tail) total-vm:7627620kB, anon-rss:7621248kB, file-rss:1408kB, shmem-rss:0kB, UID:1000 pgtables:14968kB oom_score_adj:0
```

Using ulimit: <https://ss64.com/bash/ulimit.html>

```console
ubuntu@vps-9e6a8f0e:~$ (ulimit -v 5000000 && head -c 10G /dev/zero | tail > /dev/null; echo $?)
tail: memory exhausted
1
```

This does not create a log of the OOM killer since ulimit terminated the command, not the OOM killer.

While by increasing the size enforced by ulimit, we still trigger the OOM killer:

```console
ubuntu@vps-9e6a8f0e:~$ (ulimit -v 10000000 && head -c 10G /dev/zero | tail > /dev/null ; echo $?)
137
```
