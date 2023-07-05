---
title: BTRFS
description: 
published: true
date: 2023-07-05T10:16:50.136Z
tags: 
editor: markdown
dateCreated: 2023-07-05T10:16:50.136Z
---

# BTRFS

# No space left on device

По каким то разным причинам может оказаться так что метаданные займут всю память и работа с диском будет не возможна. При этом стандартные утилиты linux показывают что есть ещё достаточное количество свободной памяти.

**Просмотр реальной свободной памяти на btrfs:**
```
sudo btrfs filesystem show /

Label: none  uuid: c98bd0f4-e67e-46b7-94d5-6930652fb5dc
	Total devices 1 FS bytes used 284.51GiB
	devid    1 size 375.69GiB used 375.69GiB path /dev/nvme0n1p5
```

Если `used` будет равно или почти равно `size`, то свободной памяти нет.

**Решение проблемы (если сработал какой-то из шагов, то к следующему переходить не нужно):**
1. Пробуем запустить балансировку, если это возможно:
   ```
   for USAGE in {10..50..10}; do
      sudo btrfs balance start -dusage=$USAGE -musage=$USAGE /your-space;
   done
   ```
2. Освобождаем память если это возможно, желательно удалить что-то большое и повторяем пункт 1;
3. Пробуем хак с изменением размера диска и повторяем пункт 1:
   ```
   sudo btrfs filesystem resize -100M /your-space; sudo btrfs filesystem resize +100M /your-space
   ```
