---
title: AsusWRT
description: 
published: true
date: 2024-08-30T18:10:26.988Z
tags: 
editor: markdown
dateCreated: 2024-08-30T17:59:46.799Z
---

# AsusWrt

# Установка entware(opkg)

```shell
# Проверяем наличие подключённой флешки
$ blkid
/dev/sda1: LABEL="Transcend" UUID="d943116e-a3f3-47a9-a910-e2c18be3d2ba"

# Размонтируем
$ umount /dev/sda1

# Форматируем в ext3
$ mkfs.ext3 /dev/sda1 -L storage

# Монтируем обратно, с именем диска
$ mkdir -p /tmp/mnt/storage
$ mount /dev/sda1 /tmp/mnt/storage

# Создаем папку asusware.arm и необходимую иерархию
$ cd /tmp/mnt/storage
$ mkdir -p asusware.arm asusware.arm/bin asusware.arm/doc asusware.arm/etc asusware.arm/include asusware.arm/lib asusware.arm/sbin asusware.arm/share asusware.arm/tmp asusware.arm/tmp/opt asusware.arm/usr asusware.arm/var asusware.arm/var/lock
$ touch asusware.arm/.asusrouter

# Перезагружаемся
$ reboot

# Устанавливаем скриптом согласно архитектуре
$ wget -O - http://bin.entware.net/armv7sf-k3.2/installer/generic.sh | sh
```

Меняем конфиг `/opt/etc/opkg.conf`.
`dest root /` нужно заменить на `dest root /tmp`.
`lists_dir ext /opt/var/opkg-lists` на `lists_dir ext /opt/lib/opkg-lists`.

```
$ vi /opt/etc/opkg.conf
```

