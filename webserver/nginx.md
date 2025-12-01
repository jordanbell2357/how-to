# ELK

## hostnamectl

```bash
sudo hostnamectl set-hostname histfile
```

## Fail2ban

```bash
sudo apt install fail2ban
sudo systemctl enable fail2ban
```

## Nginx

<https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-22-04>

```bash
sudo apt install nginx
```

```bash
sudo systemctl enable nginx
```

```bash
sudo systemctl start nginx
```

## Certbot

```bash
sudo apt install python3 python3-dev python3-venv libaugeas-dev gcc
sudo python3 -m venv /opt/certbot/
sudo /opt/certbot/bin/pip install --upgrade pip
sudo ln -s /opt/certbot/bin/certbot /usr/bin/certbot
sudo certbot --nginx
echo "0 0,12 * * * root /opt/certbot/bin/python -c 'import random; import time; time.sleep(random.random() * 3600)' && sudo certbot renew -q" | sudo tee -a /etc/crontab > /dev/null
```

## Java

<https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-on-ubuntu-22-04>

```bash
sudo apt install default-jre
```

```bash
sudo apt install default-jdk
```
