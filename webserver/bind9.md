# BIND9

https://www.zenarmor.com/docs/linux-tutorials/how-to-set-up-bind-dns-server-on-ubuntu-linux

```bash
sudo apt install bind9 bind9utils bind9-doc dnsutils -y
```


```bash
sudo vi /etc/bind/named.conf.options
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

; RS2 runs the DNS → ns1.example.com must point to RS2
ns1     IN  A   158.69.60.101

; RS1 is where example.com should resolve to
@       IN  A   148.113.200.36
www     IN  A   148.113.200.36
"/etc/bind/zones/db.example.com" 21L, 513B written
```


```
nameserver 158.69.60.101
/etc/resolv.conf
```

```
ubuntu@histfile:~$ cat /etc/hosts
127.0.0.1       localhost

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost   ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts

127.0.1.1       vps-329cc0aa.vps.ovh.ca vps-329cc0aa
```

```
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


```bash
sudo vi /etc/bind/named.conf.options
```

```
options {
    directory "/var/cache/bind";

    forwarders {
        8.8.8.8;
        8.8.4.4;
    };

    dnssec-validation auto;

    listen-on port 53 { 127.0.0.1; };
    listen-on-v6 { ::1; };

    allow-query { localhost; };
};
```



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
