---
title: Container systemd-nspawn
description: 
published: true
date: 2022-02-24T12:59:06.732Z
tags: 
editor: markdown
dateCreated: 2022-02-24T12:59:06.732Z
---

# Разворачивание systemd-nspawn контейнера
OS: Debian 10 buster

# Скачивание контейнера
Здесь название контейнера будет **DEBCONT**
```bash
sudo debootstrap \
--keyring=./key.gpg \ # Ключ для репозитория(не обязателен), нужен если ставить убунту на дебиан или другую систему.
--arch=amd64 \                 # архитектура, если отсутствует, тогда берётся архитектура хост системы.
--components main,contrib,non-free \ # сразу включаем несвободные пакеты не придётся потом править `sources.list`
--include=dbus,sudo,ssh,locales,net-tools,bash-completion \ # добавляем необходимые пакеты, чтоб не пришлось их в системе ставить
stable \                       # версия системы (stretch, buster)
/var/lib/machines/DEBCONT \    # папку для контейнера используем /var/lib/machines/, чтобы не пришлось создавать service файл
http://deb.debian.org/debian/ # адрес репозитория, если не ставить то будет использоваться системный репозиторий
```

# Настройка контейнера для запуска
1. Запуск в режиме chroot

```
sudo systemd-nspawn --machine DEBCONT
```

2. Меняем пароль у `root` командой `passwd`
3. Добавляем пользователя системы `adduser`, будет создана папка пользователя и предложение ввести пароль
4. Добавляем пользователя в группу sudo.
4. Активируем сеть в контейнере `systemctl enable systemd-networkd.service`
5. Выходим

# Дополнительные настройки контейнера
Создаём файл настроек `/etc/systemd/nspawn/DEBCONT.nspawn`. Имя файла должно совпадать с именем контейнера.
**Внимание!** Если в имени контейнера имеются символы нижнего подчёркивания, то в файле /etc/systemd/nspawn/<имя контейнера>.nspawn их нужно опустить. Например, если имя контейнера debian_8_jessie, указать debian8jessie.nspawn.

Все настройки описаны в https://www.freedesktop.org/software/systemd/man/systemd.nspawn.html

# Запуск и автозагрузка
1. Запуск контейнера осуществляется командой `sudo machinectl start DEBCONT`.  
    После этого будут созданы два сетевых интерфейса. `ve-DEBCONT@if2` - интерфейс по которому мы можем обратиться из контейнера к хост-машине. `host0@if10` - интерфейс внутри контейнера по которому мы можем обратиться из хост-машины к контейнеру.
2. Проверить состояние контейнера можно командой `sudo machinectl status DEBCONT`
    В секции `Address` будут ip адреса по которым можно обратиться к контейнеру
3. Можно соединиться через `sudo machinectl login DEBCONT`(нужен пакет dbus в контейнере). Это соединение по умолчанию отбрасывает root пользователя. Чтобы разрешить, нужно добавить устройство `/dev/pts/1` в файл `/etc/securetty`.
4. Добавить контейнер в автозагрузку можно командой `sudo systemctl enable systemd-nspawn@DEBCONT.service`

# Настройки после запуска контейнера
## Настройка локалей для корректного отображения
Выполняется командой `sudo dpkg-reconfigure locales`
В появившемся списке выберите локали `ru_RU.UTF-8` и `en_US.UTF-8` и нажмите <OK>. На следующем экране по умолчанию выберите локаль `ru_RU.UTF-8`.

## Чтобы избавиться от ошибки *unable to resolve host: Временный сбой в разрешении имен*
```
echo 127.0.1.1 `cat /etc/hostname` | sudo tee /etc/hosts
```

# Проблемы
Если мы устанавливаем контейнер debian 8 (jessie) на хост debian 10 (buster), то не будет работать команда `sudo machinectl login DEBCONT` и по умолчанию сеть не поднимется. 
## Поднять сеть в контейнере

### Проброс портов
Оставляем настройки виртуальных серверов, которые находятся вне видимости остальных участников сети и пробрасываем порты через основную систему.
Вы должны убедиться что в системе уже такие порты не заняты.
Синтаксис команды Port: `Port=<тип соединения udp,tcp, может отсутсвовать>:<порт на хост системе>:<порт в контейнере>`

```
[Network]
VirtualEthernet=yes
Port=tcp:23:22
Port=80:80
Port=443:443
```

### MACVLAN
Минусы этой технологии в том, что хост система не видит в сети контейнер, но остальные видят.

```
[Network]
MACVLAN=enp0s25
```



### Мост
1. Создаём мост в хост-системе через `systemd-networkd`  
    https://wiki.archlinux.org/index.php/Systemd-networkd#Bridge_interface
    Создаём файлы

```
/etc/systemd/network/DEBCONT-bridge.netdev
------------------------------------
[NetDev]
Name=br0
Kind=bridge
```
  
```
/etc/systemd/network/DEBCONT-bind.network
---------------------------------
[Match]
Name=en*
[Network]
Bridge=br0
```
```
/etc/systemd/network/DEBCONT-bridge.network
-------------------------------------
[Match]
Name=br0
[Network]
DHCP=ipv4
```
2. В конфигурации контейнера `/etc/systemd/nspawn/DEBCONT.nspawn`
```
[Network]
Bridge=br0
```
3. Перезапускаем контейнер и `systemd-networkd`