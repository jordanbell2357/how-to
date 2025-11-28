# ELK

## hostnamectl

```bash
sudo hostnamectl set-hostname nginx-server
```

## Fail2ban

```bash
sudo apt install fail2ban
sudo systemctl enable fail2ban
```

## Java

<https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-on-ubuntu-22-04>

```bash
sudo apt install default-jre
```

```bash
sudo apt install default-jdk
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
sudo nginx -t
sudo systemctl restart nginx
```
