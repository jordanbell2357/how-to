# Apache HTTP Server

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


