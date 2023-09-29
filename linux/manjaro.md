---
title: Manjaro
description: 
published: true
date: 2023-09-29T11:03:48.966Z
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

## The account is locked due to 3 failed attempts

Это из-за того что используется `pam_faillock.so`.
Чтобы изменить это поведение то меняем конфиг `/etc/security/faillock.conf`,
и меняем там строку `deny = 3` на требуемую, чтобы отключить нужно поставить `0`

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

## Включить DNSOverTLS в NetworkManager

Включаем работу DNS через `systemd-resolved`.

Создаём файл
`/etc/NetworkManager/conf.d/10-dns-systemd-resolved.conf`
```
[main]
dns=systemd-resolved
```

Запускаем сервисы
```
sudo systemctl enable --now systemd-resolved
sudo systemctl restart NetworkManager
```

Включение DNSOverTLS в подключении:
```
nmcli connection modify MyConnection connection.dns-over-tls 2
```

Либо в конфиге для всех подключений:
```
[connection]
connection.dns-over-tls=1
```

> "yes" (2) use DNSOverTls and disabled fallback, "opportunistic" (1) use DNSOverTls but allow fallback to unencrypted resolution, "no" (0) don't ever use DNSOverTls.

https://networkmanager.dev/docs/api/latest/nm-settings-nmcli.html

### Проверка
- В `/etc/resolv.conf` должен быть указан `127.0.0.53`.
- Проверяем что systemd-resolve работает на порту `sudo ss -lntp | grep '\(State\|:53 \)'`
- Проверяем  `resolvectl status`. Тут в конкретном подключении должен быть указан `DNSOverTLS=opportunistic` или `+DNSOverTLS`
- `resolvectl query ya.ru` должно показать `Data was acquired via local or encrypted transport: yes`
- Проверка wireshark'ом, в проходящих пакетах не должно быть протокола DNS

### Работа dns отдельно от NetworkManager

Можно включить принудительно в конфиге необходимые настройки, но если требуется, чтобы всеми настройками управлял NetworkManager, то все настройки должны быть закоментированы
`/etc/systemd/resolved.conf`
```
[Resolve]
DNS=1.1.1.1 9.9.9.9
DNSOverTLS=yes
DNSSEC=yes
FallbackDNS=8.8.8.8 1.0.0.1 8.8.4.4
```

И выключить DNS в NetworkManager

`/etc/NetworkManager/conf.d/10-dns-systemd-resolved.conf`
```
[main]
dns=systemd-resolved
systemd-resolved=false
```

# Links

https://wiki.archlinux.org/title/pacman