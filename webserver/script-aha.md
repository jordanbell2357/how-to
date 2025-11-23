# Script and aha

<https://man7.org/linux/man-pages/man1/script.1.html>

>  script - make typescript of terminal session

<https://manpages.ubuntu.com/manpages/noble/man1/aha.1.html>

>  aha — Convert ANSI escape sequences to HTML

```console
ubuntu@vps-9e6a8f0e:~$ script -q -c "journalctl -n 10 -u ssh.service --no-pager" | aha -b > journalctl_ssh.html
```

```console
ubuntu@vps-9e6a8f0e:~$ sudo rsync journalctl_ssh.html /var/www/jordanbell.org/public_html/html
```

```console
ubuntu@vps-9e6a8f0e:~$ curl -L -O https://install.speedtest.net/app/cli/ookla-speedtest-1.2.0-linux-x86_64.tgz
ubuntu@vps-9e6a8f0e:~$ tar --one-top-level -xzf ookla-speedtest-1.2.0-linux-x86_64.tgz
ubuntu@vps-9e6a8f0e:~$ cd ookla-speedtest-1.2.0-linux-x86_64
```

```console
ubuntu@vps-9e6a8f0e:~/ookla-speedtest-1.2.0-linux-x86_64$ ./speedtest
==============================================================================

You may only use this Speedtest software and information generated
from it for personal, non-commercial use, through a command line
interface on a personal computer. Your use of this software is subject
to the End User License Agreement, Terms of Use and Privacy Policy at
these URLs:

        https://www.speedtest.net/about/eula
        https://www.speedtest.net/about/terms
        https://www.speedtest.net/about/privacy

==============================================================================

Do you accept the license? [type YES to accept]: YES
License acceptance recorded. Continuing.


   Speedtest by Ookla

      Server: Teksavvy Solutions Inc - Toronto, ON (id: 32632)
         ISP: OVHcloud
Idle Latency:     7.35 ms   (jitter: 0.05ms, low: 7.31ms, high: 7.39ms)
    Download:   385.94 Mbps (data used: 184.2 MB)
                 85.10 ms   (jitter: 14.50ms, low: 7.21ms, high: 127.99ms)
      Upload:   386.04 Mbps (data used: 193.0 MB)
                 88.57 ms   (jitter: 15.83ms, low: 7.12ms, high: 126.67ms)
 Packet Loss:     0.0%
  Result URL: https://www.speedtest.net/result/c/713dc87c-5774-4a3d-a17f-fcf57d67d847
ubuntu@vps-9e6a8f0e:~/ookla-speedtest-1.2.0-linux-x86_64$ exit
exit
Script done.
ubuntu@vps-9e6a8f0e:~/ookla-speedtest-1.2.0-linux-x86_64$ cat ookla.log | aha -b > ookla.html
ubuntu@vps-9e6a8f0e:~/ookla-speedtest-1.2.0-linux-x86_64$ sudo rsync ookla.html /var/www/jordanbell.org/public_html/html
```

<https://jordanbell.org/html/journalctl_ssh.html>


```console
ubuntu@vps-9e6a8f0e:~/ookla-speedtest-1.2.0-linux-x86_64$ script stress-ng.log
Script started, output log file is 'stress-ng.log'.
ubuntu@vps-9e6a8f0e:~/ookla-speedtest-1.2.0-linux-x86_64$ sudo stress-ng --metrics-brief --oomable --timeout=30s --bigheap 10 --bigheap-growth 64m
stress-ng: info:  [28256] setting to a 30 secs run per stressor
stress-ng: info:  [28256] dispatching hogs: 10 bigheap
stress-ng: warn:  [28262] bigheap: WARNING: finished prematurely after just 2.46 secs
stress-ng: warn:  [28257] bigheap: WARNING: finished prematurely after just 2.74 secs
stress-ng: warn:  [28269] bigheap: WARNING: finished prematurely after just 2.98 secs
stress-ng: warn:  [28266] bigheap: WARNING: finished prematurely after just 3.22 secs
stress-ng: warn:  [28259] bigheap: WARNING: finished prematurely after just 3.45 secs
stress-ng: warn:  [28260] bigheap: WARNING: finished prematurely after just 3.83 secs
stress-ng: warn:  [28267] bigheap: WARNING: finished prematurely after just 4.19 secs
stress-ng: warn:  [28261] bigheap: WARNING: finished prematurely after just 4.81 secs
stress-ng: warn:  [28264] bigheap: WARNING: finished prematurely after just 5.94 secs
stress-ng: warn:  [28258] bigheap: WARNING: finished prematurely after just 9.25 secs
stress-ng: metrc: [28256] stressor       bogo ops real time  usr time  sys time   bogo ops/s     bogo ops/s
stress-ng: metrc: [28256]                           (secs)    (secs)    (secs)   (real time) (usr+sys time)
stress-ng: metrc: [28256] bigheap             344      3.78      3.76     17.15        90.92          16.45
stress-ng: info:  [28256] skipped: 0
stress-ng: info:  [28256] passed: 10: bigheap (10)
stress-ng: info:  [28256] failed: 0
stress-ng: info:  [28256] metrics untrustworthy: 0
stress-ng: info:  [28256] successful run completed in 8.75 secs
ubuntu@vps-9e6a8f0e:~/ookla-speedtest-1.2.0-linux-x86_64$ sudo dmesg | grep -i "Out of memory"
[151619.439904] Out of memory: Killed process 28274 (stress-ng-bighe) total-vm:978308kB, anon-rss:887384kB, file-rss:1024kB, shmem-rss:0kB, UID:0 pgtables:1820kB oom_score_adj:1000
[151619.649457] Out of memory: Killed process 28263 (stress-ng-bighe) total-vm:978308kB, anon-rss:903640kB, file-rss:896kB, shmem-rss:0kB, UID:0 pgtables:1852kB oom_score_adj:1000
[151619.903759] Out of memory: Killed process 28272 (stress-ng-bighe) total-vm:1109380kB, anon-rss:984152kB, file-rss:768kB, shmem-rss:0kB, UID:0 pgtables:2016kB oom_score_adj:1000
[151620.217683] Out of memory: Killed process 28270 (stress-ng-bighe) total-vm:1240452kB, anon-rss:1159896kB, file-rss:1024kB, shmem-rss:0kB, UID:0 pgtables:2356kB oom_score_adj:1000
[151620.474668] Out of memory: Killed process 28271 (stress-ng-bighe) total-vm:1502596kB, anon-rss:1427416kB, file-rss:896kB, shmem-rss:0kB, UID:0 pgtables:2880kB oom_score_adj:1000
[151620.796315] Out of memory: Killed process 28276 (stress-ng-bighe) total-vm:1764740kB, anon-rss:1679192kB, file-rss:1024kB, shmem-rss:0kB, UID:0 pgtables:3372kB oom_score_adj:1000
[151621.190653] Out of memory: Killed process 28275 (stress-ng-bighe) total-vm:2026884kB, anon-rss:1931480kB, file-rss:1024kB, shmem-rss:0kB, UID:0 pgtables:3864kB oom_score_adj:1000
[151621.774711] Out of memory: Killed process 28268 (stress-ng-bighe) total-vm:2616708kB, anon-rss:2542936kB, file-rss:896kB, shmem-rss:0kB, UID:0 pgtables:5068kB oom_score_adj:1000
[151622.793412] Out of memory: Killed process 28273 (stress-ng-bighe) total-vm:3992964kB, anon-rss:3874008kB, file-rss:1024kB, shmem-rss:0kB, UID:0 pgtables:7668kB oom_score_adj:1000
[151625.929022] Out of memory: Killed process 28265 (stress-ng-bighe) total-vm:7597444kB, anon-rss:7491672kB, file-rss:896kB, shmem-rss:0kB, UID:0 pgtables:14748kB oom_score_adj:1000
ubuntu@vps-9e6a8f0e:~/ookla-speedtest-1.2.0-linux-x86_64$ exit
exit
Script done.
ubuntu@vps-9e6a8f0e:~/ookla-speedtest-1.2.0-linux-x86_64$ cat stress-ng.log | aha -b stress-ng.html
Unknown parameter "stress-ng.html"
ubuntu@vps-9e6a8f0e:~/ookla-speedtest-1.2.0-linux-x86_64$ cat stress-ng.log | aha -b > stress-ng.html
ubuntu@vps-9e6a8f0e:~/ookla-speedtest-1.2.0-linux-x86_64$ sudo rsync stress-ng.html /var/www/jordanbell.org/pu
blic_html/html
```

<https://jordanbell.org/html/stress-ng.html>

