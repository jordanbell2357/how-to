# OVH VPS




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
``

```console
ubuntu@vps-9e6a8f0e:~$ sudo apt update
ubuntu@vps-9e6a8f0e:~$ sudo apt upgrade
```


# Ookla Speedtest CLI 

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

## Apache

<https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-22-04>

```console
ubuntu@vps-9e6a8f0e:~$ sudo apt install apache2
ubuntu@vps-9e6a8f0e:~$ sudo mkdir /var/www/histfile.org
ubuntu@vps-9e6a8f0e:~$ sudo chown -R $USER:www-data /var/www/histfile.org
ubuntu@vps-9e6a8f0e:~$ sudo chmod -R u=rwX,go=rX /var/www/histfile.org
ubuntu@vps-9e6a8f0e:~$ sudo vi /var/www/histfile.org/index.html
ubuntu@vps-9e6a8f0e:~$ sudo a2ensite histfile.org.conf
Enabling site histfile.org.
To activate the new configuration, you need to run:
  systemctl reload apache2
ubuntu@vps-9e6a8f0e:~$ sudo a2dissite 000-default.conf
Site 000-default disabled.
To activate the new configuration, you need to run:
  systemctl reload apache2
ubuntu@vps-9e6a8f0e:~$ sudo apache2ctl configtest
Syntax OK
ubuntu@vps-9e6a8f0e:~$ sudo systemctl reload apache2
```
