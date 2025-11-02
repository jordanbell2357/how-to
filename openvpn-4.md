We follow <https://openvpn.net/as-docs/tutorials/tutorial--install-ssl-certificate.html>

We use `openssl-req`. <https://docs.openssl.org/master/man1/openssl-req/>

> `-new`
> This option generates a new certificate request. It will prompt the user for the relevant field values. The actual fields prompted for and their maximum and minimum sizes
> are specified in the configuration file and any requested extensions.
>
> If the `-key` option is not given it will generate a new private key using information specified in the configuration file or given with the `-newkey` and `-pkeyopt` options,
> else by default an RSA key with 2048 bits length.

```console
openvpn@ASBuildImage-ubuntu24-v2:~$ openssl req -newkey rsa:2048 -keyout openvpn.pem -out openvpn.pem -noenc -sha256
⋮
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:New York
Locality Name (eg, city) []:New York
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Jordan Bell
Organizational Unit Name (eg, section) []:Development
Common Name (e.g. server FQDN or YOUR name) []:openvpnaccessserver
Email Address []:jordan.bell@gmail.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

