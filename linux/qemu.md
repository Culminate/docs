---
title: Qemu
description: 
published: true
date: 2023-07-04T12:54:24.793Z
tags: 
editor: markdown
dateCreated: 2023-07-04T12:06:44.326Z
---

# Qemu

# Установка на manjaro

```
yay -S qemu-desktop virt-manager virt-viewer
sudo systemctl enable --now libvirtd.service
```

# Даём права пользователю

1. Uncomment lines in `/etc/libvirt/libvirtd.conf`:
	- Set the UNIX domain socket group ownership to libvirt, (around line 85) `unix_sock_group = "libvirt"`
	- Set the UNIX socket permissions for the R/W socket (around line 102) `unix_sock_rw_perms = "0770"`
3. Add your user account to libvirt group:
	```
	sudo usermod -a -G libvirt $(whoami)
	newgrp libvirt
	sudo systemctl restart libvirtd.service
	```
  
# Запускаем сеть

`virsh net-start default` Запускаем сеть по умолчанию, после перезагруки она будет выключена.
`virsh net-autostart default` Добавляем сеть по умолчанию в автозагрузку.
`virsh net-list --all` Список сетей со статусами:

```
 Name      State      Autostart   Persistent
----------------------------------------------
 default   active       yes          yes

```


# Запуск

Запускаем `virt-manager` из консоли, либо из меню `Virtual Machine Manager`


# links

https://christitus.com/vm-setup-in-linux/
https://computingforgeeks.com/install-kvm-qemu-virt-manager-arch-manjar/
https://www.reddit.com/r/VFIO/comments/6iwth1/network_default_is_not_active_after_every/