# Pygments

<https://pygments.org/>

```bash
sudo apt install python3-pygments
```

```console
ubuntu@vps-9e6a8f0e:~$ journalctl -n 10 -u ssh.service --no-pager > journalctl.txt
```

```console
ubuntu@vps-9e6a8f0e:~$ pygmentize -l shell-session -f html -O "full,style=monokai" journalctl.txt > journalctl.html
```

```console
sudo rsync journalctl.html /var/www/jordanbell.org/public_html/html
```

<https://jordanbell.org/html/journalctl.html>
