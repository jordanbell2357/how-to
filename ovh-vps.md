# OVH VPS

## SSH key pairs

```console
ubuntu@LAPTOP-JBell:~$ ssh-keygen -t ed25519 -C "ovh-vps" -f ~/.ssh/ovh-vps
writing RSA key
```

```console
ubuntu@LAPTOP-JBell:~$ cat .ssh/ovh-vps.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIM5diIQZt8YH8JmsxT2XUUYQ1ZjA9rfzyqtkXgCQpLDY ovh-vps
```

```console
ubuntu@LAPTOP-JBell:~$ ssh -i .ssh/ovh-vps -l ubuntu 148.113.200.36
Enter passphrase for key '.ssh/ovh-vps':
You are required to change your password immediately (administrator enforced).
⋮
WARNING: Your password has expired.
You must change your password now and login again!
Changing password for ubuntu.
Current password:
New password:
Retype new password:
passwd: password updated successfully
Connection to 148.113.200.36 closed.
```

```console
ubuntu@LAPTOP-JBell:~$ ssh -i .ssh/ovh-vps -l ubuntu 148.113.200.36
```

## VPS

```console
ubuntu@vps-9e6a8f0e:~$ sudo apt update
ubuntu@vps-9e6a8f0e:~$ sudo apt upgrade
```

Basic convenience tool:

```console
ubuntu@vps-9e6a8f0e:~$ sudo apt install plocate
```

## perf

We follow <https://www.brendangregg.com/perf.html>

```console
ubuntu@vps-9e6a8f0e:~$ sudo perf stat -a sleep 5

 Performance counter stats for 'system wide':

          20008.76 msec cpu-clock                        #    4.000 CPUs utilized
               348      context-switches                 #   17.392 /sec
                16      cpu-migrations                   #    0.800 /sec
               524      page-faults                      #   26.189 /sec
   <not supported>      cycles
   <not supported>      instructions
   <not supported>      branches
   <not supported>      branch-misses

       5.002220504 seconds time elapsed
```

## openssl-speed

<https://docs.openssl.org/3.1/man1/openssl-speed/>

<https://www.feistyduck.com/library/openssl-cookbook/online/openssl-command-line/performance.html>

```console
ubuntu@vps-9e6a8f0e:~$ openssl speed rsa2048
Doing 2048 bits private rsa's for 10s: 14351 2048 bits private RSA's in 10.00s
Doing 2048 bits public rsa's for 10s: 310068 2048 bits public RSA's in 9.99s
version: 3.0.13
built on: Thu Sep 18 11:12:48 2025 UTC
options: bn(64,64)
compiler: gcc -fPIC -pthread -m64 -Wa,--noexecstack -Wall -fzero-call-used-regs=used-gpr -DOPENSSL_TLS_SECURITY_LEVEL=2 -Wa,--noexecstack -g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/build/openssl-S7huCI/openssl-3.0.13=. -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/build/openssl-S7huCI/openssl-3.0.13=/usr/src/openssl-3.0.13-0ubuntu3.6 -DOPENSSL_USE_NODELETE -DL_ENDIAN -DOPENSSL_PIC -DOPENSSL_BUILDING_OPENSSL -DNDEBUG -Wdate-time -D_FORTIFY_SOURCE=3
CPUINFO: OPENSSL_ia32cap=0xfffa3223478bffff:0x7a9
                  sign    verify    sign/s verify/s
rsa 2048 bits 0.000697s 0.000032s   1435.1  31037.8
```

```console
ubuntu@vps-9e6a8f0e:~$ nproc
4
```

Improvement using 4 cores:

```console
ubuntu@vps-9e6a8f0e:~$ openssl speed -multi $(nproc) rsa2048
⋮
                  sign    verify    sign/s verify/s
rsa 2048 bits 0.000188s 0.000009s   5315.5 117386.1
```

No improvement after using more than 4 cores:

```console
                  sign    verify    sign/s verify/s
rsa 2048 bits 0.000189s 0.000009s   5279.0 114110.2
```


## Ookla Speedtest CLI 

<https://www.speedtest.net/apps/cli>

```console
ubuntu@vps-9e6a8f0e:~$ curl -L -O https://install.speedtest.net/app/cli/ookla-speedtest-1.2.0-linux-x86_64.tgz
ubuntu@vps-9e6a8f0e:~$ tar -xzf ookla-speedtest-1.2.0-linux-x86_64.tgz
```

```console
ubuntu@vps-9e6a8f0e:~$ ./speedtest
⋮
   Speedtest by Ookla

      Server: Bell Canada - Toronto, ON (id: 53393)
         ISP: OVHcloud
Idle Latency:     8.43 ms   (jitter: 0.04ms, low: 8.34ms, high: 8.45ms)
    Download:   386.46 Mbps (data used: 323.5 MB)
                 81.48 ms   (jitter: 7.95ms, low: 8.26ms, high: 122.73ms)
      Upload:   386.00 Mbps (data used: 183.3 MB)
                 85.80 ms   (jitter: 14.81ms, low: 8.15ms, high: 130.18ms)
 Packet Loss:     0.0%
  Result URL: https://www.speedtest.net/result/c/50a8171f-7d30-424c-a123-99458ac5c434
```

