---
title: Manjaro
description: 
published: true
date: 2023-07-14T04:51:54.807Z
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

## Пересобрать все AUR приложения после обновления

Скачиваем [rebuild-detector](https://github.com/maximbaz/rebuild-detector) и передаём `yay` вывод из него.

```
yay -S rebuild-detector
yay -S --rebuild --answerclean A --answerdiff N $(checkrebuild | cut -f2)
```