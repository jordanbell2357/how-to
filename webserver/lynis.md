# Lynis

https://cisofy.com/documentation/lynis/get-started/


```console
  -[ Lynis 3.0.9 Results ]-

  Warnings (2):
  ----------------------------
  ! Found one or more vulnerable packages. [PKGS-7392]
      https://cisofy.com/lynis/controls/PKGS-7392/

  ! Found some information disclosure in SMTP banner (OS or software name) [MAIL-8818]
      https://cisofy.com/lynis/controls/MAIL-8818/

  Suggestions (55):
  ----------------------------
  * This release is more than 4 months old. Check the website or GitHub to see if there is an update available. [LYNIS]
      https://cisofy.com/lynis/controls/LYNIS/

  * Install libpam-tmpdir to set $TMP and $TMPDIR for PAM sessions [DEB-0280]
      https://cisofy.com/lynis/controls/DEB-0280/

  * Install apt-listbugs to display a list of critical bugs prior to each APT installation. [DEB-0810]
      https://cisofy.com/lynis/controls/DEB-0810/

  * Install apt-listchanges to display any significant changes prior to any upgrade via APT. [DEB-0811]
      https://cisofy.com/lynis/controls/DEB-0811/
```

...


```console
================================================================================

  Lynis security scan details:

  Hardening index : 61 [############        ]
  Tests performed : 267
  Plugins enabled : 1

  Components:
  - Firewall               [V]
  - Malware scanner        [X]

  Scan mode:
  Normal [V]  Forensics [ ]  Integration [ ]  Pentest [ ]

  Lynis modules:
  - Compliance status      [?]
  - Security audit         [V]
  - Vulnerability scan     [V]

  Files:
  - Test and debug information      : /var/log/lynis.log
  - Report data                     : /var/log/lynis-report.dat

================================================================================

  Lynis 3.0.9

  Auditing, system hardening, and compliance for UNIX-based systems
  (Linux, macOS, BSD, and others)

  2007-2021, CISOfy - https://cisofy.com/lynis/
  Enterprise support available (compliance, plugins, interface and tools)

================================================================================

  [TIP]: Enhance Lynis audits by adding your settings to custom.prf (see /etc/lynis/default.prf for all settings)
```



One of the recommendations is to install mod_evasive.

<https://www.linode.com/docs/guides/modevasive-on-apache/>

<https://www.server-world.info/en/note?os=Ubuntu_22.04&p=httpd2&f=5>


Section

```console
# Include module configuration:
Include mods-enabled/*.load
Include mods-enabled/*.conf
```

has appended


```console
<IfModule mod_evasive20.c>
    DOSHashTableSize 3097
    DOSPageCount 2
    DOSSiteCount 50
    DOSPageInterval 1
    DOSSiteInterval 1
    DOSBlockingPeriod 60
    DOSEmailNotify <jordan@jordanbell.org>
</IfModule>
```

Then we restart Apache httpd

```bash
sudo systemctl restart apache2
```

modsecurity

<https://www.digitalocean.com/community/tutorials/how-to-set-up-modsecurity-with-apache-on-ubuntu-14-04-and-debian-8>

<https://beaglesecurity.com/blog/article/modsecurity-apache-installation-guide.html>

```bash
sudo apt install libapache2-mod-security2
```

```bash
sudo cp /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf
```

We then edit modsecurity.conf. We change the line `SecRuleEngine DetectionOnly` to `SecRuleEngine On`.

```bash
sudo sed -i "s/SecRuleEngine DetectionOnly/SecRuleEngine On/" /etc/modsecurity/modsecurity.conf
sudo sed -i "s/SecResponseBodyAccess On/SecResponseBodyAccess Off/" /etc/modsecurity/modsecurity.conf
```

```bash
systemctl restart apache2
```
