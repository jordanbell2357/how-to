<https://ubuntu.com/server/docs/service-openssh>

https://kontext.tech/article/448/install-hadoop-330-on-linux

https://linuxconfig.org/ubuntu-20-04-hadoop

```bash
sudo apt-get install openssh-client openssh-server
sudo apt-get install ssh pdsh
```

https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html

```bash
sudo apt-get install default-jre openjdk-11-jre-headless openjdk-8-jre-headless openjdk-8-jdk
```

```bash
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
```

https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/FileSystemShell.html
