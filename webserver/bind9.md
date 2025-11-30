# BIND9

https://www.zenarmor.com/docs/linux-tutorials/how-to-set-up-bind-dns-server-on-ubuntu-linux

```bash
sudo apt install bind9 bind9utils bind9-doc dnsutils -y
```

```bash
sudo vi /etc/bind/named.conf.options
```

```
options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        forwarders {
        8.8.8.8;
        8.8.4.4;
        };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation auto;

        listen-on port 53 { 127.0.0.1; };
        listen-on-v6 { ::1; };

        allow-query {
        localhost;
        };
};
```


```bash
sudo vi /etc/bind/named.conf.local
```

```
//
// Do any local configuration here
//

zone "example.com" {
type master;
file "/etc/bind/zones/db.example.com";
};

zone "1.168.192.in-addr.arpa" {
type master;
file "/etc/bind/zones/db.192";
};

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";
```

```bash
sudo mkdir -p /etc/bind/zones
sudo cp /etc/bind/db.local /etc/bind/zones/db.example.com
sudo cp /etc/bind/db.127 /etc/bind/zones/db.192
```

```bash
sudo vi /etc/bind/zones/db.example.com
```

```
;
; BIND data file for local loopback interface
;
$TTL 604800
@   IN  SOA ns1.example.com. admin.example.com. (
        3        ; Serial  (increase on every edit)
        604800   ; Refresh
        86400    ; Retry
        2419200  ; Expire
        604800 ) ; Negative Cache TTL
;

@       IN  NS  ns1.example.com.

; histfile.org runs the DNS and its IP is 158.69.60.101
ns1     IN  A   158.69.60.101

; jordanbell.org is where example.com should resolve
; jordanbell.org has IP 148.113.200.36
@       IN  A   148.113.200.36
www     IN  A   148.113.200.36
```

```bash
echo "nameserver 127.0.0.1" | sudo tee /etc/resolv.conf
```

```bash
sudo systemctl disable --now systemd-resolved
```

```bash
sudo vi /etc/hosts
```

```
127.0.0.1       localhost

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost   ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts

127.0.1.1       histfile
```


```bash
sudo vi /etc/bind/named.conf.options
```

```
options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        forwarders {
        8.8.8.8;
        8.8.4.4;
        };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation auto;

        listen-on port 53 { 127.0.0.1; };
        listen-on-v6 { ::1; };

        allow-query {
        localhost;
        };
};
```


```bash
sudo systemctl restart named
```

Before restricting access to nameserver only to localhost on histfile.org:

```console
ubuntu@LAPTOP-JBell:~$ dig @158.69.60.101 bob.com

; <<>> DiG 9.18.39-0ubuntu0.24.04.2-Ubuntu <<>> @158.69.60.101 bob.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 43542
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: cff30e44c86f763101000000692c4fe8f6fe650f9f02ae84 (good)
;; QUESTION SECTION:
;bob.com.                       IN      A

;; ANSWER SECTION:
bob.com.                600     IN      CNAME   xrek5v.ckwoy.com.
xrek5v.ckwoy.com.       60      IN      A       206.119.83.34

;; Query time: 394 msec
;; SERVER: 158.69.60.101#53(158.69.60.101) (UDP)
;; WHEN: Sun Nov 30 09:08:41 EST 2025
;; MSG SIZE  rcvd: 110
```

After restricting access to the nameserver to localhost on histfile.org:

```
ubuntu@LAPTOP-JBell:~$ dig @158.69.60.101 example.com
;; communications error to 158.69.60.101#53: connection refused
;; communications error to 158.69.60.101#53: connection refused
;; communications error to 158.69.60.101#53: connection refused

; <<>> DiG 9.18.39-0ubuntu0.24.04.2-Ubuntu <<>> @158.69.60.101 example.com
; (1 server found)
;; global options: +cmd
;; no servers could be reached
```


```
ubuntu@histfile:~$ dig @127.0.0.1 example.com

; <<>> DiG 9.18.39-0ubuntu0.24.04.2-Ubuntu <<>> @127.0.0.1 example.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 44488
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 9f50ea9b6ffeac2f01000000692c515036e859e0bfd3e4eb (good)
;; QUESTION SECTION:
;example.com.                   IN      A

;; ANSWER SECTION:
example.com.            604800  IN      A       148.113.200.36

;; Query time: 1 msec
;; SERVER: 127.0.0.1#53(127.0.0.1) (UDP)
;; WHEN: Sun Nov 30 14:14:40 UTC 2025
;; MSG SIZE  rcvd: 84
```

```
ubuntu@histfile:~$ dig @127.0.0.1 -x 192.168.1.20

; <<>> DiG 9.18.39-0ubuntu0.24.04.2-Ubuntu <<>> @127.0.0.1 -x 192.168.1.20
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 35386
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 123940549e17fba501000000692cb6d1ceff33af13e1d012 (good)
;; QUESTION SECTION:
;20.1.168.192.in-addr.arpa.     IN      PTR

;; ANSWER SECTION:
20.1.168.192.in-addr.arpa. 604800 IN    PTR     www.example.com.

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1) (UDP)
;; WHEN: Sun Nov 30 21:27:45 UTC 2025
;; MSG SIZE  rcvd: 111
```


We can make our DNS server temporarily open to the internet using socat.

```bash
sudo socat UDP4-LISTEN:53,fork,bind=158.69.60.101 UDP4:127.0.0.1:53
```

Then on our laptop we succeed in using the nameserver

```
ubuntu@LAPTOP-JBell:~$ dig @158.69.60.101 example.com

; <<>> DiG 9.18.39-0ubuntu0.24.04.2-Ubuntu <<>> @158.69.60.101 example.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 59036
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 7eaafeefab39360601000000692cbe14d0decc8b63d67af7 (good)
;; QUESTION SECTION:
;example.com.                   IN      A

;; ANSWER SECTION:
example.com.            604800  IN      A       148.113.200.36

;; Query time: 12 msec
;; SERVER: 158.69.60.101#53(158.69.60.101) (UDP)
;; WHEN: Sun Nov 30 16:58:45 EST 2025
;; MSG SIZE  rcvd: 84
```

Once we terminate the socat command, using the nameserver from the internet goes back to failing:

```
ubuntu@LAPTOP-JBell:~$ dig @158.69.60.101 example.com
;; communications error to 158.69.60.101#53: connection refused
;; communications error to 158.69.60.101#53: connection refused
;; communications error to 158.69.60.101#53: connection refused

; <<>> DiG 9.18.39-0ubuntu0.24.04.2-Ubuntu <<>> @158.69.60.101 example.com
; (1 server found)
;; global options: +cmd
;; no servers could be reached
```

When socat is running again, we use the nameserver on our laptop

```console
ubuntu@LAPTOP-JBell:~$ dig +short @158.69.60.101 example.com
148.113.200.36
```

Together this is

```bash
IP_ADDRESS=$(dig +short @158.69.60.101 example.com)

curl --resolve example.com:80:$IP_ADDRESS http://example.com/
```

Then

```console
ubuntu@LAPTOP-JBell:~$ curl --resolve example.com:80:148.113.200.36 http://example.com/
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<html>
 <head>
  <title>Index of /</title>
 </head>
 <body>
<h1>Index of /</h1>
  <table>
   <tr><th valign="top"><img src="/icons/blank.gif" alt="[ICO]"></th><th><a href="?C=N;O=D">Name</a></th><th><a href="?C=M;O=A">Last modified</a></th><th><a href="?C=S;O=A">Size</a></th><th><a href="?C=D;O=A">Description</a></th></tr>
   <tr><th colspan="5"><hr></th></tr>
   <tr><th colspan="5"><hr></th></tr>
</table>
<address>Apache/2.4.58 (Ubuntu) Server at example.com Port 80</address>
</body></html>
```

