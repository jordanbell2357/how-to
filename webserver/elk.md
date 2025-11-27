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
sudo vi /etc/nginx/sites-available/histfile.org
```

```
server {
        listen 80;
        listen [::]:80;

        server_name histfile.org www.histfile.org;

        auth_basic "Restricted Access";
        auth_basic_user_file /etc/nginx/htpasswd.users;

        location / {
            proxy_pass http://localhost:5601;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/histfile.org /etc/nginx/sites-enabled/histfile.org
```

```bash
sudo nginx -t
sudo systemctl restart nginx
```

## Elasticsearch

<https://www.digitalocean.com/community/tutorials/how-to-install-elasticsearch-logstash-and-kibana-elastic-stack-on-ubuntu-22-04>

```bash
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic.gpg
```

```bash
echo "deb [signed-by=/usr/share/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
```

```bash
sudo apt update
sudo apt install elasticsearch
```

```bash
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
```

```console
ubuntu@nginx-server:~$ curl -X GET "localhost:9200"
{
  "name" : "nginx-server",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "hbee1OT5Rc-OdZs2dz0pjg",
  "version" : {
    "number" : "7.17.29",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "580aff1a0064ce4c93293aaab6fcc55e22c10d1c",
    "build_date" : "2025-06-19T01:37:57.847711500Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.3",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

## Kibana

```bash
sudo apt install kibana
sudo systemctl enable kibana
sudo systemctl start kibana
```

```bash
echo "kibana:`openssl passwd -apr1`" | sudo tee -a /etc/nginx/htpasswd.users
```

Then

<http://histfile.org>

<img width="1911" height="952" alt="image" src="https://github.com/user-attachments/assets/f658c434-65d2-4c7c-ac3b-933295207f9a" />




