---
title: AsusWRT
description: 
published: true
date: 2024-08-30T17:59:46.799Z
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

Монтируем обратно
$ mount /dev/sda1 /tmp/mnt/storage

```

