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

## stressapptest

<https://github.com/stressapptest/stressapptest>

```console
ubuntu@vps-9e6a8f0e:~$ stressapptest -W
2025/11/13-04:34:22(UTC) Log: Commandline - stressapptest -W
2025/11/13-04:34:22(UTC) Stats: SAT revision 1.0.11_autoconf, 64 bit binary
2025/11/13-04:34:22(UTC) Log: reproducible @ reproducible on Mon Apr  1 08:22:50 UTC 2024 from open source release
2025/11/13-04:34:22(UTC) Log: 1 nodes, 4 cpus.
2025/11/13-04:34:22(UTC) Log: Defaulting to 4 copy threads
2025/11/13-04:34:22(UTC) Log: Total 7751 MB. Free 7307 MB. Hugepages 0 MB. Targeting 7171 MB (92%)
2025/11/13-04:34:22(UTC) Log: Prefer plain malloc memory allocation.
2025/11/13-04:34:22(UTC) Log: Using mmap() allocation at 0x7fe5bfd00000.
2025/11/13-04:34:22(UTC) Stats: Starting SAT, 7171M, 20 seconds
2025/11/13-04:34:24(UTC) Log: Region mask: 0x1
2025/11/13-04:34:34(UTC) Log: Seconds remaining: 10
2025/11/13-04:34:45(UTC) Stats: Found 0 hardware incidents
2025/11/13-04:34:45(UTC) Stats: Completed: 739158.00M in 20.00s 36952.52MB/s, with 0 hardware incidents, 0 errors
2025/11/13-04:34:45(UTC) Stats: Memory Copy: 739158.00M at 36955.83MB/s
2025/11/13-04:34:45(UTC) Stats: File Copy: 0.00M at 0.00MB/s
2025/11/13-04:34:45(UTC) Stats: Net Copy: 0.00M at 0.00MB/s
2025/11/13-04:34:45(UTC) Stats: Data Check: 0.00M at 0.00MB/s
2025/11/13-04:34:45(UTC) Stats: Invert Data: 0.00M at 0.00MB/s
2025/11/13-04:34:45(UTC) Stats: Disk: 0.00M at 0.00MB/s
2025/11/13-04:34:45(UTC)
2025/11/13-04:34:45(UTC) Status: PASS - please verify no corrected errors
2025/11/13-04:34:45(UTC)
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

## dmidecode

```console
ubuntu@vps-9e6a8f0e:~$ sudo dmidecode --type memory
# dmidecode 3.5
Getting SMBIOS data from sysfs.
SMBIOS 2.8 present.

Handle 0x1000, DMI type 16, 23 bytes
Physical Memory Array
        Location: Other
        Use: System Memory
        Error Correction Type: Multi-bit ECC
        Maximum Capacity: 8000 MB
        Error Information Handle: Not Provided
        Number Of Devices: 1

Handle 0x1100, DMI type 17, 40 bytes
Memory Device
        Array Handle: 0x1000
        Error Information Handle: Not Provided
        Total Width: Unknown
        Data Width: Unknown
        Size: 8000 MB
        Form Factor: DIMM
        Set: None
        Locator: DIMM 0
        Bank Locator: Not Specified
        Type: RAM
        Type Detail: Other
        Speed: Unknown
        Manufacturer: QEMU
        Serial Number: Not Specified
        Asset Tag: Not Specified
        Part Number: Not Specified
        Rank: Unknown
        Configured Memory Speed: Unknown
        Minimum Voltage: Unknown
        Maximum Voltage: Unknown
        Configured Voltage: Unknown
```

```console
ubuntu@vps-9e6a8f0e:~$ sudo passwd -S
root L 2025-10-26 0 99999 7 -1
```

## Prime95

<https://www.mersenne.org/download/>

```console
ubuntu@vps-9e6a8f0e:~/p95v3019b20.linux64$ curl -L -O https://download.mersenne.ca/gimps/v30/30.19/p95v3019b20.linux64.tar.gz
```


```console
ubuntu@vps-9e6a8f0e:~/p95v3019b20.linux64$ tar -x -z --one-top-level -f p95v3019b20.linux64.tar.gz
```

```console
ubuntu@vps-9e6a8f0e:~/p95v3019b20.linux64$ cd p95v3019b20.linux64
ubuntu@vps-9e6a8f0e:~/p95v3019b20.linux64$ ./mprime
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

## /etc/ssh/sshd_config

We want to modify the contents of `/etc/ssh/sshd_config`. We can use sudo vi. Instead we do the following entirely in the shell. We
use `grep -n` to determine the line number of an entry in the sshd configuration:

```console
ubuntu@vps-9e6a8f0e:~$ sudo cat /etc/ssh/sshd_config | grep -n PasswordAuthentication
66:#PasswordAuthentication yes
88:# PasswordAuthentication.  Depending on your PAM configuration,
92:# PAM authentication, then enable this but set PasswordAuthentication
```

Then we use sed to replace this line (line 66):

```console
ubuntu@vps-9e6a8f0e:~$ sudo sed -i '66c\PasswordAuthentication no' /etc/ssh/sshd_config
```

```console
ubuntu@vps-9e6a8f0e:~$ sudo cat /etc/ssh/sshd_config | grep -n PasswordAuthentication
66:PasswordAuthentication no
88:# PasswordAuthentication.  Depending on your PAM configuration,
92:# PAM authentication, then enable this but set PasswordAuthentication
```

```console
ubuntu@vps-9e6a8f0e:~$ sudo cat /etc/ssh/sshd_config | grep -n PermitRootLogin
42:PermitRootLogin no
90:# the setting of "PermitRootLogin prohibit-password".
```

Let's use tee [^tee] to write output as root, and take this as a chance to review this pattern.

[^tee]: <https://pubs.opengroup.org/onlinepubs/000095399/utilities/tee.html>

```console
ubuntu@vps-9e6a8f0e:~$ echo bob | sudo cat > bob1; ls -l bob1
-rw-rw-r-- 1 ubuntu ubuntu 4 Nov 10 23:55 bob1
```

compared to

```console
ubuntu@vps-9e6a8f0e:~$ echo bob | sudo tee -a bob2; ls -l bob2
bob
-rw-r--r-- 1 root root 4 Nov 11 00:08 bob2
```

or to eliminate standard output

```console
ubuntu@vps-9e6a8f0e:~$ echo bob | sudo tee -a bob2 1> /dev/null; ls -l bob2
-rw-r--r-- 1 root root 4 Nov 11 00:09 bob2
```

We want a program with super user privileges to create a file, not a redirection from a Bash session being run as a regular user. We could instead use `sudo bash -c` thus

```console
ubuntu@vps-9e6a8f0e:~$ sudo bash -c "echo bob > bob3"; ls -l bob3
-rw-r--r-- 1 root root 4 Nov 11 00:19 bob3
```

```console
ubuntu@vps-9e6a8f0e:~$ echo AllowUsers ubuntu | sudo tee -a /etc/ssh/sshd_config
AllowUsers ubuntu
ubuntu@vps-9e6a8f0e:~$ tail -n 1 /etc/ssh/sshd_config
AllowUsers ubuntu
```

```console
ubuntu@vps-9e6a8f0e:~$ sudo sshd -t
```

## PAM

```console
ubuntu@vps-9e6a8f0e:~$ sudo sed -i '4c\#@include common-auth' /etc/pam.d/sshd
ubuntu@vps-9e6a8f0e:~$ cat /etc/pam.d/sshd | grep -n common-auth
4:#@include common-auth
```

```console
ubuntu@vps-9e6a8f0e:~$ cat /etc/pam.d/sshd | grep -n common-password
55:@include common-password
ubuntu@vps-9e6a8f0e:~$ sudo sed -i '55c\#@include common-password' /etc/pam.d/sshd
ubuntu@vps-9e6a8f0e:~$ cat /etc/pam.d/sshd | grep -n common-password
55:#@include common-password
```

## UFW

<http://documentation.ubuntu.com/server/how-to/security/firewalls/>

```console
ubuntu@vps-9e6a8f0e:~$ sudo ufw limit ssh/tcp
Rule added
Rule added (v6)
```

