# OpenVPN Part 2: easy-rsa

## Deploy droplet R with SSH public key EASY_RSA.pub

```console
ubuntu@LAPTOP-JBell:~$ cat .ssh/EASY_RSA.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOKivAF1DIVMFYBT0wK+kCPYmzO3YuAxWadpHG/Bws1f EASY_RSA
```

## Connect from L to R as root using SSH private key EASY_RSA

We use the private key on local machine L to connect as root to remote machine R.[^connect-droplet]

[^connect-droplet]: <https://docs.digitalocean.com/products/droplets/how-to/connect-with-ssh/openssh/>

```console
ubuntu@LAPTOP-JBell:~$ ssh -i .ssh/EASY_RSA -l root 104.131.173.60
The authenticity of host '104.131.173.60 (104.131.173.60)' can't be established.
ED25519 key fingerprint is SHA256:ghS9ATttKKWP6pRjETHUIj0skohCoI/cS8cNjLdtfAw.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

On machine R, we make a user `easyrsa`. [^adduser]

[^adduser]: https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu

```console
root@easy-rsa-s-1vcpu-1gb-35gb-intel-nyc3-01:~# adduser easyrsa
```

```console
root@easy-rsa-s-1vcpu-1gb-35gb-intel-nyc3-01:~# usermod -aG sudo easyrsa
```

Then use rsync to copy the SSH key pair EASY_RSA, EASY_RSA.pub from the root user home folder to the `easyrsa` user home folder: [^rsync]

[^rsync]: <https://download.samba.org/pub/rsync/rsync.1>

```console
root@easy-rsa-s-1vcpu-1gb-35gb-intel-nyc3-01:~# rsync --archive --chown=easyrsa:easyrsa ~/.ssh /home/easyrsa
root@easy-rsa-s-1vcpu-1gb-35gb-intel-nyc3-01:~# cat /home/easyrsa/.ssh/authorized_keys
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOKivAF1DIVMFYBT0wK+kCPYmzO3YuAxWadpHG/Bws1f EASY_RSA
```

## Connect from L to R using EASY_RSA private key

Now on machine L, we use SSH to connect as user `easyrsa` to machine R.

```console
ubuntu@LAPTOP-JBell:~$ ssh -i .ssh/EASY_RSA -l easyrsa 104.131.173.60
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.8.0-71-generic x86_64)
⋮
```

## Download easy-rsa

Now we install easy-rsa. [^easy-rsa]

[^easy-rsa]: <https://github.com/OpenVPN/easy-rsa/releases/tag/v3.2.4>

```console
easyrsa@easy-rsa-s-1vcpu-1gb-35gb-intel-nyc3-01:~$ curl -L -o EasyRSA-3.2.4.tgz https://github.com/OpenVPN/easy-rsa/releases/download/v3.2.4/EasyRSA-3.2.4.tgz
```

```console
easyrsa@easy-rsa-s-1vcpu-1gb-35gb-intel-nyc3-01:~$ tar -xzf EasyRSA-3.2.4.tgz
tar: Ignoring unknown extended header keyword 'LIBARCHIVE.xattr.com.apple.metadata:kMDItemTextContentLanguage'
```

This warning occurs because the tgz file was made using bsdtar from libarchive [^libarchive] rather than with GNU tar. We can avoid the warnings by
`sudo apt install libarchive-tools` to install bsdtar and then `bsdtar -xzf EasyRSA-3.2.4.tgz`.

Using GNU tar, we disable warnings with the option `--warning=no-$KEYWORD` where here we use `KEYWORD=unknown-keyword`. [^gnutar]

[^gnutar]: <https://www.gnu.org/software/tar/manual/html_node/warnings.html> and <https://www.gnu.org/software/tar/manual/html_node/Archive-Extraction-Warnings.html>

```console
easyrsa@easy-rsa-s-1vcpu-1gb-35gb-intel-nyc3-01:~$ tar -xzf EasyRSA-3.2.4.tgz --warning=no-unknown-keyword
```

## ufw

Set up ufw:

```console
easyrsa@easy-rsa-s-1vcpu-1gb-35gb-intel-nyc3-01:~/EasyRSA-3.2.4$ sudo ufw app list
Available applications:
  OpenSSH
easyrsa@easy-rsa-s-1vcpu-1gb-35gb-intel-nyc3-01:~/EasyRSA-3.2.4$ sudo ufw allow OpenSSH
Rules updated
Rules updated (v6)
easyrsa@easy-rsa-s-1vcpu-1gb-35gb-intel-nyc3-01:~/EasyRSA-3.2.4$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
easyrsa@easy-rsa-s-1vcpu-1gb-35gb-intel-nyc3-01:~/EasyRSA-3.2.4$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
```

## Use easy-rsa

We follow <https://community.openvpn.net/Pages/EasyRSA3-OpenVPN-Howto>

```console
easyrsa@easy-rsa-s-1vcpu-1gb-35gb-intel-nyc3-01:~/EasyRSA-3.2.4$ ./easyrsa init-pki

Notice
------
'init-pki' complete; you may now create a CA or requests.

Your newly created PKI dir is:
* /home/easyrsa/EasyRSA-3.2.4/pki

Using Easy-RSA configuration:
* undefined
```

```console
easyrsa@easy-rsa-s-1vcpu-1gb-35gb-intel-nyc3-01:~/EasyRSA-3.2.4$ ./easyrsa build-ca nopass

Enter New CA Key Passphrase:

Confirm New CA Key Passphrase:
⋮
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:

Notice
------
CA creation complete. Your new CA certificate is at:
* /home/easyrsa/EasyRSA-3.2.4/pki/ca.crt

Build-ca completed successfully.
```

## easy-rsa on machine S

From machine L, we connect to machine S using SSH:

```console
ubuntu@LAPTOP-JBell:~$ ssh -i .ssh/OPEN_VPN -l openvpn 159.65.255.114
```

As user `openvpn` in machine S,

```console
penvpn@ASBuildImage-ubuntu24-v2:~$ curl -L -o EasyRSA-3.2.4.tgz https://github.com/OpenVPN/easy-rsa/releases/download/v3.2.4/EasyRSA-3.2.4.tgz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 89360  100 89360    0     0   371k      0 --:--:-- --:--:-- --:--:--  371k
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

## Send CSR from S to R and impport

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

Notice
------
Request successfully imported with short-name: openvpn-as
This request is now ready to be signed.
```


```console
easyrsa@easy-rsa-s-1vcpu-1gb-35gb-intel-nyc3-01:~/EasyRSA-3.2.4$ ./easyrsa sign-req server openvpn-as
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.
You are about to sign the following certificate:

  Requested CN:     'openvpn-as'
  Requested type:   'server'
  Valid for:        '825' days


subject=
    commonName                = openvpn-as

Type the word 'yes' to continue, or any other input to abort.
  Confirm requested details: yes

Using configuration from /home/easyrsa/EasyRSA-3.2.4/pki/43f6efc3/temp.02
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'openvpn-as'
Certificate is to be certified until Feb  4 21:39:18 2028 GMT (825 days)

Write out database with 1 new entries
Database updated

WARNING
=======
INCOMPLETE Inline file created:
* /home/easyrsa/EasyRSA-3.2.4/pki/inline/openvpn-as.inline


Notice
------
Certificate created at:
* /home/easyrsa/EasyRSA-3.2.4/pki/issued/openvpn-as.crt
```


 ```console
easyrsa@easy-rsa-s-1vcpu-1gb-35gb-intel-nyc3-01:~/EasyRSA-3.2.4$ cat pki/ca.crt
-----BEGIN CERTIFICATE-----
MIIDSzCCAjOgAwIBAgIUY6v2tbVY6w4is77XnwUVE9d5LS0wDQYJKoZIhvcNAQEL
BQAwFjEUMBIGA1UEAwwLRWFzeS1SU0EgQ0EwHhcNMjUxMTAxMjAzOTM1WhcNMzUx
MDMwMjAzOTM1WjAWMRQwEgYDVQQDDAtFYXN5LVJTQSBDQTCCASIwDQYJKoZIhvcN
AQEBBQADggEPADCCAQoCggEBAPkbWf2Mx9H5SHZd0+2CwFvIl78BMQyn6rCU/WTm
2J9WH0OZE9CY3LAWd864ibBkYXbTNAqbeJXENCUer5ZXJRRnOZxTN8t36f2P4UBn
4CY70LPfu4eYNbUY7JCMIhplH8im9gYM6RdDwzf7wsBZTLyvvikvzwkH4M3NJ5nl
mFCXQQMgUvL6A8yLbi/jKJ6yCMlz1kIcVisgiG5AH80D8ubS1gJp4RHm4wYzoi3s
2nIfvTZ6ucMDVOVNlMZvunBTaCHxMS5O46HVuCQCVqpbZYVqf5rd15XTM2LZyu+l
S0lKZDdmixhgiL/9jdRj0zi6HOUz5tdv4tDzP307ZWd6W78CAwEAAaOBkDCBjTAM
BgNVHRMEBTADAQH/MB0GA1UdDgQWBBSsKZVE1Y0Uuums4BH+GixjvxQmpjBRBgNV
HSMESjBIgBSsKZVE1Y0Uuums4BH+GixjvxQmpqEapBgwFjEUMBIGA1UEAwwLRWFz
eS1SU0EgQ0GCFGOr9rW1WOsOIrO+158FFRPXeS0tMAsGA1UdDwQEAwIBBjANBgkq
hkiG9w0BAQsFAAOCAQEAGC/OkOXNFrIeO90lBMUQTp7K9TQRRY6rbD4n6DGZNM41
/vJY1muwtxJn1RWhXu+RaNtkI2MohJGF+xNkr9I3omT+gzK84gtu9d2ce2AHtJs2
uY2GnEbYDPpkcJVZ1f30dpsInPZ5G4Kce018rQZo0OcAq6I9JZLa9AQmW6B159V7
e5OcVzxQXqJrYksbOnOToP77E6dQfggNpuTEqdvSEhpxhk7Xg0CoVJMCdOSIQ3NQ
LcC2+iQ+q1+Z7K1DKvb8osyDk9Xytxsc49dMtzX9ok0oFmW4M0eiK2DNHtV2rEyj
jdfO6ArtQ3NpSLQ/5ost3GEfc8soDcFmUxqR7j4Ljw==
-----END CERTIFICATE-----
```

In S, we now make a file `/tmp/ca.crt` with the same content. [^csr]

[^csr]: <https://www.digitalocean.com/community/tutorials/how-to-set-up-and-configure-a-certificate-authority-ca-on-debian-10>

```console
openvpn@ASBuildImage-ubuntu24-v2:~/EasyRSA-3.2.4$ vi /tmp/ca.crt
openvpn@ASBuildImage-ubuntu24-v2:~/EasyRSA-3.2.4$ sudo cp /tmp/ca.crt /usr/local/share/ca-certificates/
openvpn@ASBuildImage-ubuntu24-v2:~/EasyRSA-3.2.4$ sudo update-ca-certificates
Updating certificates in /etc/ssl/certs...
rehash: warning: skipping ca-certificates.crt,it does not contain exactly one certificate or CRL
1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
```
