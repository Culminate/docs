---
title: Qemu
description: 
published: true
date: 2023-09-27T10:26:58.659Z
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
  
# Настройка подключения к системному сокету

По умолчанию `virsh` подключается к пользовательской сессии `qemu:///session`, а она не позвляет управлять сетевыми подключениями. Можно использовать несколько представленных путей:
- Запускать через `sudo virsh`
- Указывать системный сокет через аргумент `virsh --connect qemu:///system`
- Редактировать системный конфиг `/etc/libvirt/libvirt.conf` и указать там `uri_default = "qemu:///system"`
- Редактировать пользовательский конфиг `~/.config/libvirt/libvirt.conf` и указать там `uri_default = "qemu:///system"`
- Указать переменную окружения `LIBVIRT_DEFAULT_URI=qemu:///system`

# Запускаем сеть

`sudo virsh net-start default` Запускаем сеть по умолчанию, после перезагруки она будет выключена.
`sudo virsh net-autostart default` Добавляем сеть по умолчанию в автозагрузку.
`sudo virsh net-list --all` Список сетей со статусами:

```
 Name      State      Autostart   Persistent
----------------------------------------------
 default   active       yes          yes

```

# Запуск

Запускаем `virt-manager` из консоли, либо из меню `Virtual Machine Manager`

# Полезное
## Преобразование vmdk (vbox, vmware) образа в qow2 (qemu)

Если это `.ova`, то распаковываем его `tar -xvf appliance.ova`

Преобразуем
```
qemu-img convert -O qcow2 box-disk.vmdk box-disk.qcow2
```

Создаём новую виртуальную машину и импортируем диск.

## Установить фиксированный dhcp адрес для виртуальной машины

Ищем макадрес вирутальной машины

```
sudo virsh dumpxml  $VM_NAME | grep 'mac address'
```

Выбираем необходимую сеть и редактируем её, например `default`
```
sudo virsh net-list
sudo virsh net-edit $NETWORK_NAME
```

### Вариант 1

```
sudo virsh net-update default add ip-dhcp-host "<host mac='52:54:00:00:00:01' name='bob' ip='192.168.122.45' />" --live --config
```

### Вариант 2

Находим секцию dhcp и добавляем туда `<host mac='' name='' ip=''/>`, например:

```
<dhcp>
  <range start='192.168.122.100' end='192.168.122.254'/>
  <host mac='52:54:00:6c:3c:01' name='vm1' ip='192.168.122.11'/>
  <host mac='52:54:00:6c:3c:02' name='vm2' ip='192.168.122.12'/>
  <host mac='52:54:00:6c:3c:03' name='vm3' ip='192.168.122.12'/>
</dhcp>
```

Перезапустить виртуальную сеть

```
sudo virsh net-destroy $NETWORK_NAME
sudo virsh net-start $NETWORK_NAME
```

## Используемые пути

- `/etc/libvirt/`
  - `/etc/libvirt/libvirt.conf` - клиент
  - `/etc/libvirt/libvirtd.conf` - сервер
  - `/etc/libvirt/storage/` - хранилища
  - `/etc/libvirt/qemu/` - конфиги виртуальных машин
  - `/etc/libvirt/qemu/networks` - сети
- `/var/lib/libvirt/`
  - `/var/lib/libvirt/images` - хранилище дисков виртуальных машин по умолчанию
  - `/var/lib/libvirt/qemu` - хранилище временной информации (снапшоты, снимки RAM)

# Links

https://wiki.libvirt.org
https://christitus.com/vm-setup-in-linux/
https://computingforgeeks.com/install-kvm-qemu-virt-manager-arch-manjar/
https://serverfault.com/questions/803283/how-do-i-list-virsh-networks-without-sudo