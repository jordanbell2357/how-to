# OpenVPN Part 2: easy-rsa

## Download easy-rsa to L

Now we install easy-rsa. [^easy-rsa]

[^easy-rsa]: <https://github.com/OpenVPN/easy-rsa/releases/tag/v3.2.4>

```console
ubuntu@LAPTOP-JBell:~$ curl -L -o EasyRSA-3.2.4.tgz https://github.com/OpenVPN/easy-rsa/releases/download/v3.2.4/EasyRSA-3.2.4.tgz
```

```console
ubuntu@LAPTOP-JBell:~$ tar -xzf EasyRSA-3.2.4.tgz
tar: Ignoring unknown extended header keyword 'LIBARCHIVE.xattr.com.apple.metadata:kMDItemTextContentLanguage'
```

This warning occurs because the tgz file was made using bsdtar from libarchive [^libarchive] rather than with GNU tar. We can avoid the warnings by
`sudo apt install libarchive-tools` to install bsdtar and then `bsdtar -xzf EasyRSA-3.2.4.tgz`.

Using GNU tar, we disable warnings with the option `--warning=no-$KEYWORD` where here we use `KEYWORD=unknown-keyword`. [^gnutar]

[^gnutar]: <https://www.gnu.org/software/tar/manual/html_node/warnings.html> and <https://www.gnu.org/software/tar/manual/html_node/Archive-Extraction-Warnings.html>

```console
ubuntu@LAPTOP-JBell:~$ tar -xzf EasyRSA-3.2.4.tgz --warning=no-unknown-keyword
```

## Use easy-rsa on L

We follow <https://community.openvpn.net/Pages/EasyRSA3-OpenVPN-Howto>

```console
ubuntu@LAPTOP-JBell:~/EasyRSA-3.2.4$ ./easyrsa init-pki

Notice
------
'init-pki' complete; you may now create a CA or requests.

Your newly created PKI dir is:
* /home/ubuntu/EasyRSA-3.2.4/pki

Using Easy-RSA configuration:
* undefined
```

```console
ubuntu@LAPTOP-JBell:~/EasyRSA-3.2.4$ ./easyrsa build-ca

Enter New CA Key Passphrase:

Confirm New CA Key Passphrase:
⋮
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:

Notice
------
CA creation complete. Your new CA certificate is at:
* /home/ubuntu/EasyRSA-3.2.4/pki/ca.crt

Build-ca completed successfully.
```

```console
ubuntu@LAPTOP-JBell:~/EasyRSA-3.2.4$ ./easyrsa build-server-full openvpnaccessserver nopass
⋮
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'openvpnaccessserver'
Certificate is to be certified until Feb  5 03:02:47 2028 GMT (825 days)

Write out database with 1 new entries
Data Base Updated

WARNING
=======
INCOMPLETE Inline file created:
* /home/ubuntu/EasyRSA-3.2.4/pki/inline/private/openvpnaccessserver.inline


Notice
------
Certificate created at:
* /home/ubuntu/EasyRSA-3.2.4/pki/issued/openvpnaccessserver.crt
```

```console
ubuntu@LAPTOP-JBell:~/EasyRSA-3.2.4$ ./easyrsa build-client-full linux-localhost nopass
⋮
Notice
------
Certificate created at:
* /home/ubuntu/EasyRSA-3.2.4/pki/issued/linux-localhost.crt
```

```console
ubuntu@LAPTOP-JBell:~/EasyRSA-3.2.4$ ./easyrsa build-client-full windows-localhost nopass
⋮
Notice
------
Certificate created at:
* /home/ubuntu/EasyRSA-3.2.4/pki/issued/windows-localhost.crt
```

## Download and use easy-rsa on S

```console
openvpn@ASBuildImage-ubuntu24-v2:~$ curl -L -o EasyRSA-3.2.4.tgz https://github.com/OpenVPN/easy-rsa/releases/download/v3.2.4/EasyRSA-3.2.4.tgz
openvpn@ASBuildImage-ubuntu24-v2:~$ tar -xzf EasyRSA-3.2.4.tgz --warning=no-unknown-keyword
```

```console
openvpn@ASBuildImage-ubuntu24-v2:~/EasyRSA-3.2.4$ ./easyrsa init-pki

Notice
------
'init-pki' complete; you may now create a CA or requests.

Your newly created PKI dir is:
* /home/openvpn/EasyRSA-3.2.4/pki

Using Easy-RSA configuration:
* undefined
```

```console
openvpn@ASBuildImage-ubuntu24-v2:~/EasyRSA-3.2.4$ ./easyrsa gen-req openvpnaccessserver nopass
⋮
Common Name (eg: your user, host, or server name) [openvpnaccessserver]:

Notice
------
Private-Key and Public-Certificate-Request files created.
Your files are:
* req: /home/openvpn/EasyRSA-3.2.4/pki/reqs/openvpnaccessserver.req
* key: /home/openvpn/EasyRSA-3.2.4/pki/private/openvpnaccessserver.key
```


## Windows localhost

We use [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/) and make `windows-localhost`
and ``windows-localhost.ppk`.

We use
[DigiCert Certificate Utility for Windows](https://www.digicert.com/support/tools/certificate-utility-for-windows)
with Commom Name "windows-localhost" and save to `windows-localhost.csr`.

```
-----BEGIN NEW CERTIFICATE REQUEST-----
MIICwDCCAagCAQAwezELMAkGA1UEBhMCVVMxETAPBgNVBAgTCE5ldyBZb3JrMREw
DwYDVQQHEwhOZXcgWW9yazEUMBIGA1UECxMLRGV2ZWxvcG1lbnQxFDASBgNVBAoT
C0pvcmRhbiBCZWxsMRowGAYDVQQDExF3aW5kb3dzLWxvY2FsaG9zdDCCASIwDQYJ
KoZIhvcNAQEBBQADggEPADCCAQoCggEBANpcdOfJLkuhPXyAcbQphehKjvcv9iK+
jMmzEW9mPVatXZqQ8lMlC9KbsCdIjfejQ8A/hRfCEIfD5K07DaJeBn4Utc9cVTxd
uG6I4lNq/lkNg26WBGRik4y9LFtLXAYhS+Ie74jo7tWVVF7qtyM4zotYyYcrajuZ
yx4N5XzJI9MoC7nwb6zvUnoFDvlvhfi2/oIJ9A4ms7+e/Vvxq0spgGf0OWFwuxab
7otoIxQam8g5z41TFfwCmaku+YN7W/d3TnzG39HW8rQ5BQ4RjpxTLZmfXWB2Qdfq
xDv9VL5WUQEqbrn/E0CdeAad7x/lVmmgO4haUwI9R96hMMT3Jeyhwf0CAwEAAaAA
MA0GCSqGSIb3DQEBBQUAA4IBAQBkPgm5ic0+NjA5MT1WeIYCUjudEwOJFC28ZEXW
zlRpCL/NRo22vUl5z38LDCX6ftaWtIyiOSEA0rktBgW9nv45aQ9IrI3y5FKgyYMi
WaC4yXr6n2btQjBhgXzXdbD2A5k7elqvCUhLqdHefL3/gVwZRUIeMadu6qPMEgRI
QvNXWB+em+KmFVs1+yeWqy7iIVQ0bxi+EvtAtxpcv3+NEg5ykS1gKHrasGHDdZtk
EwwkyjuOPCjK1b/Pez6k/qH57+Qw6QjMo5aYgydFtC5Q0jD8tm29/vXEV+fDuEp6
no0fEJhDYL2GY1QJGKgpfdGwvkUKMdZAiIQy/wVGvEF39eap
-----END NEW CERTIFICATE REQUEST-----
```


## Easy-TLS on machine S

```console
openvpn@ASBuildImage-ubuntu24-v2:~/EasyRSA-3.2.4$ curl -L -O https://github.com/TinCanTech/easy-tls/archive/refs/tags/v2.7.0.tar.gz
openvpn@ASBuildImage-ubuntu24-v2:~/EasyRSA-3.2.4$ tar -xzf v2.7.0.tar.gz
openvpn@ASBuildImage-ubuntu24-v2:~/EasyRSA-3.2.4$ cp easy-tls-2.7.0/easytls ./
```



## easy-rsa on machine S

From machine L, we connect to machine S using SSH:

```console
ubuntu@LAPTOP-JBell:~$ ssh -i .ssh/OPEN_VPN -l openvpn 159.65.255.114
```

As user `openvpn` in machine S,

```console
penvpn@ASBuildImage-ubuntu24-v2:~$ curl -L -o EasyRSA-3.2.4.tgz https://github.com/OpenVPN/easy-rsa/releases/download/v3.2.4/EasyRSA-3.2.4.tgz
⋮
openvpn@ASBuildImage-ubuntu24-v2:~$ tar -xzf EasyRSA-3.2.4.tgz --warning=no-unknown-keyword
openvpn@ASBuildImage-ubuntu24-v2:~$ cd EasyRSA-3.2.4/
openvpn@ASBuildImage-ubuntu24-v2:~/EasyRSA-3.2.4$ ./easyrsa init-pki

Notice
------
'init-pki' complete; you may now create a CA or requests.

Your newly created PKI dir is:
* /home/openvpn/EasyRSA-3.2.4/pki

Using Easy-RSA configuration:
* undefined
```

Following <https://community.openvpn.net/Pages/EasyRSA3-OpenVPN-Howto>,

```console
openvpn@ASBuildImage-ubuntu24-v2:~/EasyRSA-3.2.4$ ./easyrsa gen-req openvpn-as nopass
⋮
Notice
------
Private-Key and Public-Certificate-Request files created.
Your files are:
* req: /home/openvpn/EasyRSA-3.2.4/pki/reqs/openvpn-as.req
* key: /home/openvpn/EasyRSA-3.2.4/pki/private/openvpn-as.key
```

This creates the folloiwng file:

```console
openvpn@ASBuildImage-ubuntu24-v2:~/EasyRSA-3.2.4$ realpath pki/reqs/openvpn-as.req
/home/openvpn/EasyRSA-3.2.4/pki/reqs/openvpn-as.req
```

## Send CSR from S to R and import

We view the certificate signing request

```console
openvpn@ASBuildImage-ubuntu24-v2:~/EasyRSA-3.2.4$ cat pki/reqs/openvpn-as.req
-----BEGIN CERTIFICATE REQUEST-----
MIICWjCCAUICAQAwFTETMBEGA1UEAwwKb3BlbnZwbi1hczCCASIwDQYJKoZIhvcN
AQEBBQADggEPADCCAQoCggEBAMHfZ712QfVQup1KhRkRE+usJTYL0X6SlJdw12RD
q3gzXysvEo1rPhJrXZ/se8uuYiR0ddjLiQpKAycIOTnOFy6GyRrFea4V/hbs7+n2
sZIzouI60vlw20RevXOSSVocsMJSJdGM9TpRCQDEPiiTvSg+m9pHPJNLvzaSSw+H
9h/zFY6e5YHKqy4Cr7sGUenV5QDAKEt8rn0+B1ueYXEssPgEQhkvJPMxOcHhMkTt
Xa/hnzrZTr273GEJHmDRKzPbWXTb1t2sdMA9y3qzAqSGzDSDDojUAYnUk2fhPR9i
4JMPDSrykTsg1Q0QaaQb9e5IY3z2u/BD1hsdiwRv1Aymi8UCAwEAAaAAMA0GCSqG
SIb3DQEBCwUAA4IBAQC2NLOFqgrmuEikrDv7stbTX8FNfE4d6D1hirbIFAumD3cO
RL0V91OZRIYPftf5Fki0egM2tqZSU/QbCFp9MISs400x5y3IV25KphTtYy/rPxQn
ggx98mtdZxPhzYKA6iKh0KkthBv0Mjp820Ier4kkR6KFgmSyuw5ZgxO2D7yD6fT3
ZyASzSITK3VC2opJJ0RDG4ecej91yuMNxNUva3+gtUVfC+w05hKfNA3AyLU6+/dq
imfJZ7sZn5wYfcumsMmm65dOU+dQV8gonmhS6Jc665vSgQqGf/YOoDABEPFzNLJi
2wyvMnHhuMZkLzfwwAGEY9C1hOiWXJsauovcZVkJ
-----END CERTIFICATE REQUEST-----
```

We make a file `openvpn-as.req` in  R with the same content, at path `/home/easyrsa/EasyRSA-3.2.4/reqs/openvpn-as.req`.

```console
easyrsa@easy-rsa-s-1vcpu-1gb-35gb-intel-nyc3-01:~/EasyRSA-3.2.4$ ./easyrsa import-req reqs/openvpn-as.req openvpn-as
⋮
Notice
------
Request successfully imported with short-name: openvpn-as
This request is now ready to be signed.
```


```console
easyrsa@easy-rsa-s-1vcpu-1gb-35gb-intel-nyc3-01:~/EasyRSA-3.2.4$ ./easyrsa sign-req server openvpn-as


Notice
------
Certificate created at:
* /home/easyrsa/EasyRSA-3.2.4/pki/issued/openvpn-as.crt
```

We now make a file `/tmp/ca.crt` in S with the same content as `/home/easyrsa/EasyRSA-3.2.4/pki/issued/openvpn-as.crt` in R. [^csr]

[^csr]: <https://www.digitalocean.com/community/tutorials/how-to-set-up-and-configure-a-certificate-authority-ca-on-debian-10>

```console
openvpn@ASBuildImage-ubuntu24-v2:~/EasyRSA-3.2.4$ vi /tmp/openvpn-as.crt # with content of openvpn-as.crt in R
openvpn@ASBuildImage-ubuntu24-v2:~/EasyRSA-3.2.4$ sudo cp /tmp/openvpn-as.crt /usr/local/share/ca-certificates/openvpn-as.crt
openvpn@ASBuildImage-ubuntu24-v2:~/EasyRSA-3.2.4$ sudo update-ca-certificates
Updating certificates in /etc/ssl/certs...
rehash: warning: skipping ca-certificates.crt,it does not contain exactly one certificate or CRL
1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
```

## Inspect openvpn-as.pem in S

In S,

```console
openvpn@ASBuildImage-ubuntu24-v2:~/EasyRSA-3.2.4$ openssl x509 -in /etc/ssl/certs/openvpn-as.pem -noout -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            d3:ac:66:4c:1e:06:44:d0:ec:98:7a:83:58:10:ce:72
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = Easy-RSA CA
        Validity
            Not Before: Nov  1 21:39:18 2025 GMT
            Not After : Feb  4 21:39:18 2028 GMT
        Subject: CN = openvpn-as
        Subject Public Key Info:
⋮
```

## DH Generation

```console
openvpn@ASBuildImage-ubuntu24-v2:~/EasyRSA-3.2.4$ ./easyrsa gen-dh
Generating DH parameters, 2048 bit long safe prime
⋮
DH parameters appear to be ok.

Notice
------

DH parameters of size 2048 created at:
* /home/openvpn/EasyRSA-3.2.4/pki/dh.pem
```

