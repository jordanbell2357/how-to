# OpenVPN exploration

OpenVPN Access Server at 64.225.5.143

<img width="1080" height="694" alt="image" src="https://github.com/user-attachments/assets/d9acee05-2bc2-4157-bd78-33b217d786d8" />

Connect to this server using OpenVPN Connect client on Windows.

We run Ookla Speedtest. [^ookla]

[^ookla]: <https://www.speedtest.net/>

<img width="386" height="595" alt="ookla" src="https://github.com/user-attachments/assets/f266bbf1-0666-445c-95f2-85fda735ecf8" />

We filter in Wireshark using IP address

```
ip.addr == 64.225.5.143
```

 <img width="1032" height="687" alt="image" src="https://github.com/user-attachments/assets/530a54f8-d845-4801-9875-b0c9ce14d7cf" />

OpenVPN uses port 1194 for TCP and for UDP. [^ports] We filter using this port:

[^openvpn]: <https://wiki.wireshark.org/OpenVPN>


In our case,  it is using UDP port 1194.

```
udp.port == 1194
```



