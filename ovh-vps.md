# OVH VPS

<https://docs.openssl.org/3.2/man1/openssl-genpkey/>



```console
ubuntu@LAPTOP-JBell:~/keys$ openssl genrsa -out openvpn.pem 2048
ubuntu@LAPTOP-JBell:~/keys$ openssl rsa -in openvpn.pem -outform PEM -pubout -out openvpn.pem.pub
writing RSA key
```



```console

```
