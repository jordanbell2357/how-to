# Certbot

<https://certbot.eff.org/instructions?ws=apache&os=pip>

<https://www.digitalocean.com/community/tutorials/how-to-use-certbot-standalone-mode-to-retrieve-let-s-encrypt-ssl-certificates-on-ubuntu-20-04>

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


## Verifying

<https://www.warp.dev/terminus/openssl-check-certificate>

```console
ubuntu@LAPTOP-JBell:~$ openssl s_client -connect histfile.org:443
CONNECTED(00000003)
depth=2 C = US, O = Internet Security Research Group, CN = ISRG Root X1
verify return:1
depth=1 C = US, O = Let's Encrypt, CN = E7
verify return:1
depth=0 CN = histfile.org
verify return:1
---
Certificate chain
 0 s:CN = histfile.org
   i:C = US, O = Let's Encrypt, CN = E7
   a:PKEY: id-ecPublicKey, 256 (bit); sigalg: ecdsa-with-SHA384
   v:NotBefore: Nov  4 06:23:18 2025 GMT; NotAfter: Feb  2 06:23:17 2026 GMT
 1 s:C = US, O = Let's Encrypt, CN = E7
   i:C = US, O = Internet Security Research Group, CN = ISRG Root X1
   a:PKEY: id-ecPublicKey, 384 (bit); sigalg: RSA-SHA256
   v:NotBefore: Mar 13 00:00:00 2024 GMT; NotAfter: Mar 12 23:59:59 2027 GMT
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIDljCCAx2gAwIBAgISBlShbYRFSNgyLGWF2IYt227IMAoGCCqGSM49BAMDMDIx
CzAJBgNVBAYTAlVTMRYwFAYDVQQKEw1MZXQncyBFbmNyeXB0MQswCQYDVQQDEwJF
NzAeFw0yNTExMDQwNjIzMThaFw0yNjAyMDIwNjIzMTdaMBcxFTATBgNVBAMTDGhp
c3RmaWxlLm9yZzBZMBMGByqGSM49AgEGCCqGSM49AwEHA0IABLLIKcDH5aXkMJLW
+bWad6R4UK4kMOtQ3b8bgQ1XWgMAH7G50f9iAbzuXpHHItUxX1f56SKoSYeq/N80
biSNtB+jggIsMIICKDAOBgNVHQ8BAf8EBAMCB4AwHQYDVR0lBBYwFAYIKwYBBQUH
AwEGCCsGAQUFBwMCMAwGA1UdEwEB/wQCMAAwHQYDVR0OBBYEFMAGGh26A+Voi4x0
cMKxrb6IhPj3MB8GA1UdIwQYMBaAFK5IntyHHUSgb9qi5WB0BHjCnACAMDIGCCsG
AQUFBwEBBCYwJDAiBggrBgEFBQcwAoYWaHR0cDovL2U3LmkubGVuY3Iub3JnLzAp
BgNVHREEIjAgggxoaXN0ZmlsZS5vcmeCEHd3dy5oaXN0ZmlsZS5vcmcwEwYDVR0g
BAwwCjAIBgZngQwBAgEwLQYDVR0fBCYwJDAioCCgHoYcaHR0cDovL2U3LmMubGVu
Y3Iub3JnLzczLmNybDCCAQQGCisGAQQB1nkCBAIEgfUEgfIA8AB1AMs49xWJfISh
RF9bwd37yW7ymlnNRwppBYWwyxTDFFjnAAABmk2+TUIAAAQDAEYwRAIgEX4cQs1Z
p78g313ybmHKQHlzu70maGkOwVTes1YHbigCIBNURqbNjF233s3u4OrGvi0ytga9
uMfnkWiXyGD5P0EsAHcADleUvPOuqT4zGyyZB7P3kN+bwj1xMiXdIaklrGHFTiEA
AAGaTb5NRAAABAMASDBGAiEAgR1j1V7S3C3NxdFoVQG/UiR1xtcwyKjOZ08FL7jZ
z/UCIQD9bW7WVZ/vd0A/Gvwh5GqymB7RvZZqcRxCyGig3a1D6DAKBggqhkjOPQQD
AwNnADBkAjAatuZ5wruJDtgxxxgKdBcUJGN4vz3azyCTvKra96x9vfEwD5/2Jvvx
tA4VBuKaVD0CMCDwws/yz2mtmC7gLzPrUdLGRJIn1mKcbB8xwX3vR0XSTQMDx4rJ
sj2Dp5ILZ+R/GQ==
-----END CERTIFICATE-----
subject=CN = histfile.org
issuer=C = US, O = Let's Encrypt, CN = E7
---
No client certificate CA names sent
Peer signing digest: SHA256
Peer signature type: ECDSA
Server Temp Key: X25519, 253 bits
---
SSL handshake has read 2416 bytes and written 394 bytes
Verification: OK
---
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
Server public key is 256 bit
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
Early data was not sent
Verify return code: 0 (ok)
---
---
Post-Handshake New Session Ticket arrived:
SSL-Session:
    Protocol  : TLSv1.3
    Cipher    : TLS_AES_256_GCM_SHA384
    Session-ID: 9C2A2E2C3141341F41968C83D99AC5C1740118FAA5F1023E09F2865349A14BF0
    Session-ID-ctx:
    Resumption PSK: 23000479098ED2AB177BE5808A431692154EDE0A2C60423987BF7394A5320A173D274E224684CC154E68545ED45DD809
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 300 (seconds)
    TLS session ticket:
    0000 - 54 0d 32 21 7d bd 55 ed-e1 1e 3f 5c 42 98 b3 be   T.2!}.U...?\B...
    0010 - 7f 31 37 8b 9a 09 b0 3b-be a3 c1 c8 07 bb a6 56   .17....;.......V

    Start Time: 1762241797
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
    Extended master secret: no
    Max Early Data: 0
---
read R BLOCK
---
Post-Handshake New Session Ticket arrived:
SSL-Session:
    Protocol  : TLSv1.3
    Cipher    : TLS_AES_256_GCM_SHA384
    Session-ID: F6A45291D67BDDF69C605BCC3AFD39AA5CB9254D3DE415E80B51B7534DA6C267
    Session-ID-ctx:
    Resumption PSK: 0AEE3DDF55071FE8967EF8ECB091153015E75AC8BAE166F269817F9F7EFF4F4BDA037DA657E502D5EC53C9904B14E856
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 300 (seconds)
    TLS session ticket:
    0000 - 97 ec e7 03 b9 e1 be df-a5 f2 3d e5 fd 54 f8 e4   ..........=..T..
    0010 - 31 c3 3b 5d c6 0f 8a ab-21 b4 5e 58 b8 b8 25 1b   1.;]....!.^X..%.

    Start Time: 1762241797
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
    Extended master secret: no
    Max Early Data: 0
---
read R BLOCK

HTTP/1.1 400 Bad Request
Date: Tue, 04 Nov 2025 07:36:47 GMT
Server: Apache/2.4.58 (Ubuntu)
Content-Length: 305
Connection: close
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>400 Bad Request</title>
</head><body>
<h1>Bad Request</h1>
<p>Your browser sent a request that this server could not understand.<br />
</p>
<hr>
<address>Apache/2.4.58 (Ubuntu) Server at histfile.org Port 443</address>
</body></html>
closed
```


## cron

```console
ubuntu@vps-9e6a8f0e:~$ echo "0 0,12 * * * root /opt/certbot/bin/python -c 'import random; import time; time.sleep(random.random() * 3600)' && sudo certbot renew -q" | sudo tee -a /etc/crontab > /dev/null
```
