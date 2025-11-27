# ELK

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
sudo systemctl enable nginx
```

```bash
sudo mkdir -p /var/www/histfile.org/html
sudo chown -R $USER:$USER /var/www/histfile.org/html
sudo chmod -R 755 /var/www/histfile.org
```

```bash
vi /var/www/histfile.org/html/index.html
```

```html
<html>
    <head>
        <title>Welcome to histfile.org!</title>
    </head>
    <body>
        <h1>Success!  The histfile.org server block is working!</h1>
    </body>
</html>
```

```bash
sudo vi /etc/nginx/sites-available/histfile.org
```


```
server {
        listen 80;
        listen [::]:80;

        root /var/www/histfile.org/html;
        index index.html index.htm index.nginx-debian.html;

        server_name histfile.org www.histfile.org;

        location / {
                try_files $uri $uri/ =404;
        }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/histfile.org /etc/nginx/sites-enabled/
```

```bash
sudo vi /etc/nginx/nginx.conf
```

Change line 23, `# server_names_hash_bucket_size 64;` to `server_names_hash_bucket_size 64;`.

```bash
sudo nginx -t
sudo systemctl restart nginx
```

## Fail2ban

```bash
sudo apt install fail2ban
sudo systemctl enable fail2ban
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
sudo systemctl start elasticsearch
sudo systemctl enable elasticsearch
```

```console
ubuntu@vps-329cc0aa:~$ curl -X GET "localhost:9200"
{
  "name" : "vps-329cc0aa",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "djJK8OAkQ8mhY08M9ZOs1A",
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
echo "kibanauser:`openssl passwd -apr1`" | sudo tee -a /etc/nginx/htpasswd.users
```

```bash
sudo vi /etc/nginx/sites-available/kibana.histfile.org
```

```
server {
    listen 80;

    server_name kibana.histfile.org;

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
sudo ln -s /etc/nginx/sites-available/kibana.histfile.org /etc/nginx/sites-enabled/kibana.histfile.org
```

```bash
sudo nginx -t
sudo systemctl reload nginx
```

<img width="1916" height="688" alt="image" src="https://github.com/user-attachments/assets/b3311ef2-eac0-499a-9668-52a51aa30a54" />

## Certbot

```bash
sudo apt install python3 python3-dev python3-venv libaugeas-dev gcc
```

```bash
sudo python3 -m venv /opt/certbot/
sudo /opt/certbot/bin/pip install --upgrade pip
```

```bash
sudo /opt/certbot/bin/pip install certbot certbot-nginx
sudo ln -s /opt/certbot/bin/certbot /usr/bin/certbot
```

```bash
echo "0 0,12 * * * root /opt/certbot/bin/python -c 'import random; import time; time.sleep(random.random() * 3600)' && sudo certbot renew -q" | sudo tee -a /etc/crontab > /dev/null
```
