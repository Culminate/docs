# Raspberry pi 4
Raspbian OS 64bit
kernel 

## Wireguard

Install

```
echo -ne "deb http://deb.debian.org/debian buster-backports main contrib non-free" | sudo tee /etc/apt/sources.list.d/buster-backports.list
sudo apt update
sudo apt install wireguard-dkms wireguard-tools

```

check

```
sudo dkms status
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIxMjc1MzUzOTEsMzA3MjU1NjI3LDMzOD
gxMzk0NF19
-->