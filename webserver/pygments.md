# Pygments

<https://pygments.org/>

```bash
sudo apt install python3-pygments
```

```console
ubuntu@LAPTOP-JBell:~$ cat session.txt
ubuntu@LAPTOP-JBell:~$ echo bob
bob
```

```console
ubuntu@LAPTOP-JBell:~$ pygmentize -l shell-session -f html -O "full,style=monokai" session.txt > session.html
```


<span style="color: #ff4689; font-weight: bold">ubuntu@LAPTOP-JBell:~$ </span><span style="color: #f8f8f2">echo </span>bob
<span style="color: #66d9ef">bob</span>
