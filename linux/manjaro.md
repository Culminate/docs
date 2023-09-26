---
title: Manjaro
description: 
published: true
date: 2023-09-26T07:50:25.199Z
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

## Не видит аудиокарту (нет звука)

Установить `sof-firmware`

# Полезное

## Удалить приложение со всеми зависимостями и конфигами

```
yay -Rcnsu <приложение>
```

## Очистить кэш yay

```
yay -Scc
```

## Отобразить установленные AUR приложения

```
yay -Qqem
```

## Найти пакет по файлу

```
yay -Qo <file> # Ищет в установленных приложениях
yay -F <file> # Ищет во всех пакетах репозитория
yay -Fx <regex> # Ищет во всех пакетах репозитория по регулярному выражению
```

## Пересобрать все AUR приложения после обновления

Скачиваем [rebuild-detector](https://github.com/maximbaz/rebuild-detector) и передаём `yay` вывод из него.

```
yay -S rebuild-detector
yay -S --rebuild --answerclean A --answerdiff N $(checkrebuild | cut -f2)
```

P.S. после установки rebuild-detector уже стоит хук на pacman но работает он в ограниченном количестве случаев.

# Links

https://wiki.archlinux.org/title/pacman