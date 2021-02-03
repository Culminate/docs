# Raspberry pi 4
Raspbian OS 64bit
kernel 

## Wireguard

Install

```
echo -ne "deb http://deb.debian.org/debian buster-backports main contrib non-free" | sudo tee /etc/apt/sources.list.d/buster-backports.list
sudo apt update
sudo apt install wireguard-dkms wireguard-tools
sudo reboot
```

check install

```
sudo dkms status
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTE1MDY3MTgwMCwzMDcyNTU2MjcsMzM4OD
EzOTQ0XX0=
-->