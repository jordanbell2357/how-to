# WireGuard

https://www.digitalocean.com/community/tutorials/how-to-set-up-wireguard-on-ubuntu-22-04


https://support.keenetic.com/hero/kn-1011/en/21427-connecting-to-a-wireguard-vpn-from-windows.html


https://www.wireguard.com/install/


https://www.wireguardconfig.com/

```bash
sudo apt update
sudo apt install wireguard
```

```bash
wg genkey | sudo tee /etc/wireguard/private.key
```

```console
<base64_server_private_key>
```

```bash
sudo chmod go= /etc/wireguard/private.key
```

```bash
sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
```

```console
3/6vqif1839TEH16Az4jsfuHGoqYwudjSP+y/uUSHVk=
```

```bash
sudo vi /etc/wireguard/wg0.conf
```

```
[Interface]
PrivateKey = <base64_server_private_key>
Address = 10.8.0.1/24
ListenPort = 51820
SaveConfig = true
```

```bash
sudo vi /etc/sysctl.conf
```

We uncomment (line 28 in my configuration)

```console
# Uncomment the next line to enable packet forwarding for IPv4
net.ipv4.ip_forward=1
```

We determine the network interface

```bash
ip route list default
```

```console
default via 158.69.60.1 dev ens3 proto dhcp src 158.69.60.101 metric 100
```

Thus the network interface is ens3.

We create the file `wg0.conf`

```bash
sudo vi /etc/wireguard/wg0.conf
```

with the following contents

```console
[Interface]
PrivateKey = <base64_server_private_key>
Address = 10.8.0.1/24
ListenPort = 51820
SaveConfig = true

PostUp = ufw route allow in on wg0 out on ens3
PostUp = iptables -t nat -I POSTROUTING -o ens3 -j MASQUERADE
PreDown = ufw route delete allow in on wg0 out on ens3
PreDown = iptables -t nat -D POSTROUTING -o ens3 -j MASQUERADE

PostUp = ip rule add table 200 from 158.69.60.101
PostUp = ip route add table 200 default via 158.69.60.101
PreDown = ip rule delete table 200 from 158.69.60.101
PreDown = ip route delete table 200 default via 158.69.60.101
```

We use resolvectl [^resolvectl] to find the IP address of the DNS server used by the interface ens3.

[^resolvectl]: <https://www.freedesktop.org/software/syst1emd/man/latest/resolvectl.html>

```bash
resolvectl dns ens3
```

```
Link 2 (ens3): 213.186.33.99
```

Now we use the public key of the Windows client, which is 3Rm2j4hlEk98AbF5cfmBHW9fXfKsLeO2UfuCbRu1jSM=


```bash
sudo wg set wg0 peer 3Rm2j4hlEk98AbF5cfmBHW9fXfKsLeO2UfuCbRu1jSM= allowed-ips 10.8.0.2
```

Works now!

<img width="987" height="647" alt="image" src="https://github.com/user-attachments/assets/9c622033-a0e9-4783-942e-b62be03fb324" />


Check connection.

## DNS leaks

https://dnsleaktest.org/

https://dnsleaktest.org/dns-leak-test/d73977a9-fa2c-4ec5-95ad-51a086750749

<img width="1443" height="897" alt="image" src="https://github.com/user-attachments/assets/62b15af2-d0ca-4cc0-ba5b-c8d7e524eae2" />

> One of the myths is that you would need a VPN Provider to prevent DNS Leak. This is not true! Unfortunately, most Leak tests are from VPN Providers or their affiliates, which leads to this intentional misconception. If you followed our guides above, you can still prevent a leak. Even the fact that some VPN Providers do have leaking DNS Settings makes things worse for you. Most people who care about such Leaks are aware of VPN for their privacy, which leads to these myths. But you do not need a VPN to take care of your DNS Privacy. A good VPN makes things better for you because you would not need to care for local DNS Settings. But at times, your VPN is down or your disconnect, your DNS might still leak. So it would be best if you first took care of your local issues then protected yourself with a VPN as a secondary privacy shield.


## CloudFlare

https://speed.cloudflare.com/

<img width="1584" height="674" alt="image" src="https://github.com/user-attachments/assets/2ec2842c-0b8e-4259-9676-0f91496c5105" />


<img width="1560" height="746" alt="image" src="https://github.com/user-attachments/assets/1e75a503-f456-4d22-b596-af8bafc1cccf" />


## BrowserLeaks

https://browserleaks.com/ip

<img width="1187" height="878" alt="row-1-column-1 (4)" src="https://github.com/user-attachments/assets/2d909a63-767f-40db-82dd-aefd6b8b4ab8" />


## DNS

https://cmdns.dev.dns-oarc.net/

<img width="717" height="370" alt="image" src="https://github.com/user-attachments/assets/505b2eaf-7095-4d5d-8ae5-f9db5e853d28" />



## Buffer Bloat

https://www.waveform.com/tools/bufferbloat?test-id=6234e80a-8070-4153-96ef-97586920ad9a

<img width="1442" height="410" alt="image" src="https://github.com/user-attachments/assets/879ea8d7-aa04-40d1-9aa5-d230bb639c84" />

