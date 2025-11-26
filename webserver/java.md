# Java

<https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-on-ubuntu-22-04>

```bash
sudo apt install default-jre
```

```bash
sudo apt install default-jdk
```

```console
ubuntu@vps-329cc0aa:~$ sudo update-alternatives --config java
There is 1 choice for the alternative java (providing /usr/bin/java).

  Selection    Path                                         Priority   Status
------------------------------------------------------------
* 0            /usr/lib/jvm/java-21-openjdk-amd64/bin/java   2111      auto mode
  1            /usr/lib/jvm/java-21-openjdk-amd64/bin/java   2111      manual mode

Press <enter> to keep the current choice[*], or type selection number:
```

```console
ubuntu@vps-329cc0aa:~$ echo JAVA_HOME="/usr/lib/jvm/java-21-openjdk-amd64" | sudo tee -a /etc/environment
JAVA_HOME="/usr/lib/jvm/java-21-openjdk-amd64"
```

```console
ubuntu@vps-329cc0aa:~$ cat /etc/environment
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin"
JAVA_HOME="/usr/lib/jvm/java-21-openjdk-amd64"
```

```console
ubuntu@vps-329cc0aa:~$ source /etc/environment
ubuntu@vps-329cc0aa:~$ echo $JAVA_HOME
/usr/lib/jvm/java-21-openjdk-amd64/bin/java
```
