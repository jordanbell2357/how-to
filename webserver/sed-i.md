# sshd


We want to modify the contents of `/etc/ssh/sshd_config`. We can use sudo vi. Instead we do the following entirely in the shell. We
use `grep -n` to determine the line number of an entry in the sshd configuration:

```console
ubuntu@vps-9e6a8f0e:~$ sudo cat /etc/ssh/sshd_config | grep -n PasswordAuthentication
66:#PasswordAuthentication yes
88:# PasswordAuthentication.  Depending on your PAM configuration,
92:# PAM authentication, then enable this but set PasswordAuthentication
```

Then we use sed to replace this line (line 66):

```console
ubuntu@vps-9e6a8f0e:~$ sudo sed -i '66c\PasswordAuthentication no' /etc/ssh/sshd_config
```

```console
ubuntu@vps-9e6a8f0e:~$ sudo cat /etc/ssh/sshd_config | grep -n PasswordAuthentication
66:PasswordAuthentication no
88:# PasswordAuthentication.  Depending on your PAM configuration,
92:# PAM authentication, then enable this but set PasswordAuthentication
```

```console
ubuntu@vps-9e6a8f0e:~$ sudo cat /etc/ssh/sshd_config | grep -n PubkeyAuthentication
47:#PubkeyAuthentication yes
ubuntu@vps-9e6a8f0e:~$ sudo sed -i '47c\PubkeyAuthentication yes' /etc/ssh/sshd_config
ubuntu@vps-9e6a8f0e:~$ sudo cat /etc/ssh/sshd_config | grep -n PubkeyAuthentication
47:PubkeyAuthentication yes
```


and

```console

```
