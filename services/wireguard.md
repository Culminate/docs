---
title: wireguard
description: 
published: true
date: 2022-02-24T12:50:48.630Z
tags: 
editor: markdown
dateCreated: 2022-02-24T12:50:48.630Z
---

# Wireguard VPN
https://www.cyberciti.biz/faq/debian-10-set-up-wireguard-vpn-server/
https://github.com/complexorganizations/wireguard-manager/blob/main/wireguard-server.sh
https://wiki.archlinux.org/index.php/WireGuard
https://christine.website/blog/site-to-site-wireguard-part-1-2019-04-02
OS: Debian 10 buster
Wireguard: v1.0.2

## Wireguard для Windows
https://www.wireguard.com/install/

Ключи генерируются в интерфейсе.

## Настройка репозитория

Чтобы установить необходимо добавить backports репозиторий

```bash
echo 'deb http://deb.debian.org/debian buster-backports main contrib non-free' | sudo tee /etc/apt/sources.list.d/buster-backports.list
```

Обновим кэш и установим `wireguard`

## Настройка сервера

### Генерация ключей

```bash
sudo -i
cd /etc/wireguard/
umask 077; wg genkey | tee privatekey | wg pubkey > publickey
```

### Редактируем файл конфигурации сервера
https://jlk.fjfi.cvut.cz/arch/manpages/man/wg.8#CONFIGURATION_FILE_FORMAT
https://jlk.fjfi.cvut.cz/arch/manpages/man/wg-quick.8#CONFIGURATION

Нам понадобится сгенерировать ключи на сервере и клиентах.
`Address` адрес с каким будут общаться пиры VPN. DHCP отсутствует в wireguard.
`AllowedIPs` тут указывается с каким ip может подключиться клиент. Важно чтобы маска была 32, если будет например 24, то больше никто из этой подсети не сможет подключиться к серверу.
`PresharedKey` необязательный параметр, ключ симметричного шифрования хендшейка.

`/etc/wireguard/wg0.conf`
```ini
[Interface]
Address = 10.8.0.1/24
ListenPort = 51820
PrivateKey = qLPcmxDXSDPnQ0xJEs7xfugphlwfKq6f+cnoFjlEqHI=

[Peer]
PublicKey = t0rlEcyng9KxTNvkW2ErLmU0lGFxb6ucONII2RsshS4=
PresharedKey = gyx3iPnd4nMaQsw7oTtiwUqdbfyuSUxuZadzCRwQGVg=
AllowedIPs = 10.8.0.2/32
```

### Маршрутизация
Если нам требуется маршрутизация, то её необходимо включить

```bash
echo "net.ipv4.ip_forward=1" | sudo tee /etc/sysctl.d/wireguard.conf
echo "net.ipv6.conf.all.forwarding=1" | sudo tee -a /etc/sysctl.d/wireguard.conf # для ipv6
sudo sysctl -p /etc/sysctl.d/wireguard.conf
```

`PreUp`, `PostUp`, `PreDown`, `PostDown` команды в секции `[Interface]` позволяют запустить скрипты в оболочке bash. А специальный символ `%i` указывает на интерфейс VPN

Например, применение правил iptables и отмена их после отключения VPN.
```ini
[Interface]
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE;
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE;
...
```

### Редактируем файл конфигурации клиента
`Address` ваш адрес в vpn сети, уточняйте у администратора
`AllowedIPs` указывается по каким маршрутам будет идти трафик. `0.0.0.0/0` - означает что весь трафик пойдёт через VPN. Сети можно указывать через запятую.
`Endpoint` это внешний ip адрес сервера
`PersistentKeepalive` интервал посылки Keepalive пакетов, нужен чтобы NAT не потерял порт.
`PresharedKey` необязательный параметр, ключ симметричного шифрования хендшейка.

```ini
[Interface]
Address = 15.15.0.2/24
PrivateKey = IDl+pGidN+9ch4T9olRr2Nzz3m6/DYxGMprTRBg4IEk=
DNS = 15.15.0.1

[Peer]
PublicKey = nTymyKMitJYgFwygzPHEhNV1s/rGNm9nunM5otB1J1w=
PresharedKey = gyx3iPnd4nMaQsw7oTtiwUqdbfyuSUxuZadzCRwQGVg=
Endpoint = 192.168.1.154:51820
AllowedIPs = 15.15.0.0/24, 192.168.1.0/24
PersistentKeepalive = 25
```

## Запуск и остановка

```bash
wg-quick up wg0   # запуск осуществляется по названию файла в /etc/wireguard/
wg-quick down wg0 # остановка
```

Так же запуск можно осуществлять с помощью systemctl.
Добавляем в автозапуск.
```bash
sudo systemctl enable --now wg-quick@wg0.service
```

## Проверка работы

`wg show` покажет основную информацию о сервере и пирах.

## Работа wireguard в network manager

После настройки нигде в GUI не реализовано включение VPN. Можно либо включить автоконнект, либо воспользоваться CLI утилитой.

```bash
nmcli connection up wireguard_connect
```

## Android DNS

Андроид по умолчанию не подхватывает DNS сервер из VPN, необходимо пробросить в VPN гугловский DNS `AllowedIPs = 8.8.8.8/32`