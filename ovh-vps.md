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

## Apache

<https://httpd.apache.org/docs/2.4/vhosts/name-based.html>

<https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-20-04>

<https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-22-04>

```console
ubuntu@vps-9e6a8f0e:~$ sudo apt install apache2
```

```console
ubuntu@vps-9e6a8f0e:~$ sudo mkdir -p /var/www/histfile.org/public_html
ubuntu@vps-9e6a8f0e:~$ sudo chown -R $USER:www-data /var/www/histfile.org/public_html
ubuntu@vps-9e6a8f0e:~$ sudo chmod -R 755 /var/www/histfile.org/public_html
```

We make a basic `index.html` file at `/var/www/histfile.org/public_html/index.html` with these contents:[^html]

[^html]: <https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/Basic_HTML_syntax#anatomy_of_an_html_document>

```html
<!doctype html>
<html lang="en-US">
  <head>
    <meta charset="utf-8" />
    <title>My test page</title>
  </head>
  <body>
    <p>This is my page</p>
  </body>
</html>
```

Here are two ways of making this file. One way uses `sudo vi`:

```console
ubuntu@vps-9e6a8f0e:~$ sudo vi /var/www/histfile.org/public_html/index.html
```

We paste the contents and save. This creates

```console
ubuntu@vps-9e6a8f0e:~$ stat /var/www/histfile.org/public_html/index.html
  File: /var/www/histfile.org/public_html/index.html
  Size: 170             Blocks: 8          IO Block: 4096   regular file
Device: 8,1     Inode: 525397      Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2025-11-03 07:13:16.094688203 +0000
Modify: 2025-11-03 07:09:03.119678526 +0000
Change: 2025-11-03 07:09:03.119678526 +0000
 Birth: 2025-11-03 07:09:03.119678526 +0000
```

Another way is longer but can all be done in the shell.
To make an `index.html` file we use a "here doc":[^here-doc]

[^here-doc]: <https://tldp.org/LDP/abs/html/here-docs.html>

```console
ubuntu@vps-9e6a8f0e:~$ cat > index.html <<EOF
> <!doctype html>
<html lang="en-US">
  <head>
    <meta charset="utf-8" />
    <title>My test page</title>
  </head>
  <body>
    <p>This is my page</p>
  </body>
</html>
> EOF
```

This file is located at

```console
ubuntu@vps-9e6a8f0e:~$ realpath index.html
/home/ubuntu/index.html
```

with this metadata

```console
ubuntu@vps-9e6a8f0e:~$ stat index.html
  File: index.html
  Size: 170             Blocks: 8          IO Block: 4096   regular file
Device: 8,1     Inode: 505         Links: 1
Access: (0664/-rw-rw-r--)  Uid: ( 1000/  ubuntu)   Gid: ( 1000/  ubuntu)
Access: 2025-11-04 04:43:56.720296542 +0000
Modify: 2025-11-04 04:43:56.722296545 +0000
Change: 2025-11-04 04:43:56.722296545 +0000
 Birth: 2025-11-04 04:43:56.720296542 +0000
```

We use rsync to copy the file and change its permissions[^rsync]

[^rsync]: <https://download.samba.org/pub/rsync/rsync.1>

```console
ubuntu@vps-9e6a8f0e:~$ sudo rsync index.html /var/www/histfile.org/public_html
```

The metadata is

```console
ubuntu@vps-9e6a8f0e:~$ stat /var/www/histfile.org/public_html/index.html
  File: /var/www/histfile.org/public_html/index.html
  Size: 170             Blocks: 8          IO Block: 4096   regular file
Device: 8,1     Inode: 525110      Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2025-11-04 04:44:22.271337245 +0000
Modify: 2025-11-04 04:44:22.271337245 +0000
Change: 2025-11-04 04:44:22.271337245 +0000
 Birth: 2025-11-04 04:44:22.271337245 +0000
```

We create a file `/etc/apache2/sites-available/histfile.org.conf`. This can be done look with `index.html`, using `vi`
on the one hand and using shell commands entirely on the other.

We make the file

```console
sudo vi /etc/apache2/sites-available/histfile.org.conf
```

with the contents

```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName histfile.org
    ServerAlias www.histfile.org
    DocumentRoot /var/www/histfile.org/public_html
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Now we use a2ensite and a2dissite.[^a2ensite]

[^a2ensite]: <https://linuxcommandlibrary.com/man/a2ensite> and <https://manpages.ubuntu.com/manpages/focal/man8/a2ensite.8.html>

> a2ensite is a fundamental utility script prevalent on Debian-based Linux distributions, such as Ubuntu, designed to efficiently manage Apache HTTP Server virtual host configurations. Its primary function is to enable a specific virtual host. The command achieves this by creating a symbolic link from the configuration file located in the `/etc/apache2/sites-available/` directory to the `/etc/apache2/sites-enabled/` directory. This symlink instructs Apache to include the linked configuration when it processes its active sites.

We use the basename of the virtual host configuration file:

```console
ubuntu@vps-9e6a8f0e:~$ basename /etc/apache2/sites-available/histfile.org.conf
histfile.org.conf
```

This virtual host configuration is enabled with

```console
ubuntu@vps-9e6a8f0e:~$ sudo a2ensite histfile.org.conf
Enabling site histfile.org.
To activate the new configuration, you need to run:
  systemctl reload apache2
```

The default virtual host configuration file is disabled.[^a2dissite]

[^a2dissite]: <https://linuxcommandlibrary.com/man/a2dissite>

> *a2dissite* is a script used to disable Apache virtual host configurations. It operates by removing the symbolic link from the **/etc/apache2/sites-enabled/** directory that points to the configuration file in **/etc/apache2/sites-available/**. This effectively tells Apache to no longer load that specific site configuration upon restart or reload. This command simplifies the management of Apache virtual hosts, making it easy to turn sites on or off without manually manipulating symbolic links. After running *a2dissite*, the Apache web server must be reloaded or restarted for the changes to take effect. It typically requires root privileges to execute.

```console
ubuntu@vps-9e6a8f0e:~$ sudo a2dissite 000-default.conf
Site 000-default disabled.
To activate the new configuration, you need to run:
  systemctl reload apache2
```

```console
ubuntu@vps-9e6a8f0e:~$ sudo apache2ctl configtest
Syntax OK
```

Then restart Apache web server using apachectl: [^apachectl]

[^apachectl]: <https://httpd.apache.org/docs/2.4/stopping.html>

```console
ubuntu@vps-9e6a8f0e:~$ sudo apachectl -k restart
```

## DNS records

Cloudflare: we make these DNS records on Cloudflare:

```
;;
;; Domain:     histfile.org.
;; Exported:   2025-11-04 07:18:55
;;
;; This file is intended for use for informational and archival
;; purposes ONLY and MUST be edited before use on a production
;; DNS server.  In particular, you must:
;;   -- update the SOA record with the correct authoritative name server
;;   -- update the SOA record with the contact e-mail address information
;;   -- update the NS record(s) with the authoritative name servers for this domain.
;;
;; For further information, please consult the BIND documentation
;; located on the following website:
;;
;; http://www.isc.org/
;;
;; And RFC 1035:
;;
;; http://www.ietf.org/rfc/rfc1035.txt
;;
;; Please note that we do NOT offer technical support for any use
;; of this zone data, the BIND name server, or any other third-party
;; DNS software.
;;
;; Use at your own risk.
;; SOA Record
histfile.org	3600	IN	SOA	dilbert.ns.cloudflare.com. dns.cloudflare.com. 2051390733 10000 2400 604800 3600

;; NS Records
histfile.org.	86400	IN	NS	dilbert.ns.cloudflare.com.
histfile.org.	86400	IN	NS	mina.ns.cloudflare.com.

;; A Records
histfile.org.	1	IN	A	148.113.200.36 ; cf_tags=cf-proxied:false
www.histfile.org.	1	IN	A	148.113.200.36 ; cf_tags=cf-proxied:false

;; AAAA Records
histfile.org.	1	IN	AAAA	2607:5300:205:200::2dec ; cf_tags=cf-proxied:false
www.histfile.org.	1	IN	AAAA	2607:5300:205:200::2dec ; cf_tags=cf-proxied:false
```

```console
ubuntu@vps-9e6a8f0e:~$ curl http://histfile.org/
<!doctype html>
<html lang="en-US">
  <head>
    <meta charset="utf-8" />
    <title>My test page</title>
  </head>
  <body>
    <p>This is my page</p>
  </body>
</html>
```

## Certbot

<https://www.digitalocean.com/community/tutorials/how-to-use-certbot-standalone-mode-to-retrieve-let-s-encrypt-ssl-certificates-on-ubuntu-20-04>

<https://certbot.eff.org/instructions?ws=apache&os=snap>

```console
ubuntu@vps-9e6a8f0e:~$ sudo apt install python3 python3-dev python3-venv libaugeas-dev gcc
ubuntu@vps-9e6a8f0e:~$ sudo python3 -m venv /opt/certbot/
ubuntu@vps-9e6a8f0e:~$ sudo /opt/certbot/bin/pip install --upgrade pip
ubuntu@vps-9e6a8f0e:~$ sudo /opt/certbot/bin/pip install certbot certbot-apache
ubuntu@vps-9e6a8f0e:~$ sudo ln -s /opt/certbot/bin/certbot /usr/bin/certbot
```


```console
ubuntu@vps-9e6a8f0e:~$ sudo certbot --apache
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Enter email address or hit Enter to skip.
 (Enter 'c' to cancel): jordan.bell@gmail.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at:
https://letsencrypt.org/documents/LE-SA-v1.5-February-24-2025.pdf
You must agree in order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y
Account registered.

Which names would you like to activate HTTPS for?
We recommend selecting either all domains, or all domains in a VirtualHost/server block.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: histfile.org
2: www.histfile.org
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 1 2
Requesting a certificate for histfile.org and www.histfile.org

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/histfile.org/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/histfile.org/privkey.pem
This certificate expires on 2026-02-02.
These files will be updated when the certificate renews.

Deploying certificate
Successfully deployed certificate for histfile.org to /etc/apache2/sites-available/histfile.org-le-ssl.conf
Successfully deployed certificate for www.histfile.org to /etc/apache2/sites-available/histfile.org-le-ssl.conf
Congratulations! You have successfully enabled HTTPS on https://histfile.org and https://www.histfile.org

NEXT STEPS:
- The certificate will need to be renewed before it expires. Certbot can automatically renew the certificate in the background, but you may need to take steps to enable that functionality. See https://certbot.org/renewal-setup for instructions.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```


## cron

```console
ubuntu@vps-9e6a8f0e:~$ echo "0 0,12 * * * root /opt/certbot/bin/python -c 'import random; import time; time.sleep(random.random() * 3600)' && sudo certbot renew -q" | sudo tee -a /etc/crontab > /dev/null
```
