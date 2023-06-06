---
title: Manjaro
description: 
published: true
date: 2023-06-06T08:59:36.003Z
tags: 
editor: markdown
dateCreated: 2023-06-06T08:03:23.678Z
---

# Manjaro

# Проблемы

## Не отображаются флаги переключателя языков

Установить `iso-flag-png`

## Долго запускаются приложения

Удалить `xdg-desktop-portal-gnome`

# Полезное

## Удалить приложение со всеми зависимостями и конфигами

```
yay -Rcnsu <приложение>
```

## Отобразить установленные AUR приложения

```
yay -Qqem
```