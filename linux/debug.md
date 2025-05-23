---
title: Debug
description: 
published: 1
date: 2025-05-23T20:21:31.524Z
tags: 
editor: markdown
dateCreated: 2025-05-20T17:10:34.258Z
---

# Debug

Есть несколько способов собрать и связать дебаг информацию в linux.

## Самый простой

После создания исполняемого файла, можно скопировать debug информацию в отдельный файл с помощью утилиты [objcopy](https://man7.org/linux/man-pages/man1/objcopy.1.html).
Создать `.gnu_debuglink` секцию в исполняемом файле с именем debug файла и его crc32 суммой. С помощью этой секции отладчик находит debug файл.
И удалить debug информацию из исполняемого файла.

```
objcopy --only-keep-debug executable executable.debug
objcopy --add-gnu-debuglink=executable.debug executable
objcopy --strip-debug executable
```

## build-id

`.note.gnu.build-id`

# links

https://gcc.gnu.org/wiki/DebugFission
https://sourceware.org/gdb/current/onlinedocs/gdb.html/Separate-Debug-Files.html
https://sourceware.org/gdb/current/onlinedocs/gdb.html/Index-Files.html
https://interrupt.memfault.com/blog/gnu-build-id-for-firmware