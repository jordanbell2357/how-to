# WireGuard

https://www.digitalocean.com/community/tutorials/how-to-set-up-wireguard-on-ubuntu-22-04


https://support.keenetic.com/hero/kn-1011/en/21427-connecting-to-a-wireguard-vpn-from-windows.html

```bash
sudo apt install wireguard
```

```bash
wg genkey | sudo tee /etc/wireguard/private.key
sudo chmod go= /etc/wireguard/private.key
```

```bash
sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
```

...

```bash

```

```bash

```

```bash

```


https://www.wireguard.com/install/
