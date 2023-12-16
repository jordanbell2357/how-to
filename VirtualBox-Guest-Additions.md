# VirtualBox Guest Additions

https://www.virtualbox.org/wiki/Downloads

https://download.virtualbox.org/virtualbox/7.0.12/VirtualBox-7.0.12-159484-Win.exe

https://download.virtualbox.org/virtualbox/7.0.12/Oracle_VM_VirtualBox_Extension_Pack-7.0.12.vbox-extpack

Run VirtualBox as Administrator to install the following:

![image](https://github.com/jordanbell2357/how-to/assets/47544607/6eb9bc97-c114-463e-a303-e65b535917d8)

Then run VirtualBox as regular user.

![image](https://github.com/jordanbell2357/how-to/assets/47544607/63b82067-0cce-4b93-90a5-1b68ad2043e1)

```bash
cd /media/ubuntu/VBox_GAs_7.0.12
```
```bash
sudo ./VBoxLinuxAdditions.run
```
![image](https://github.com/jordanbell2357/how-to/assets/47544607/4651a106-6df0-474b-ba89-52031b1f4423)

For me, this now enables shared clipboard between Windows and VirtualBox VM. Before doing these steps, this option can be selected shared clipboard does not function.
