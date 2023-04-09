---
title: ansible
description: 
published: true
date: 2023-04-09T16:13:49.865Z
tags: 
editor: markdown
dateCreated: 2023-04-09T12:11:06.065Z
---

# Ansible
Средство автоматизации множества хостов по ssh.

OS: Debian 11

Работает только на linux, несмотря на то что это python.

# Установка 
## Tabs {.tabset}
### python
Можно установить с помощью pip
Представлены 3 варианта

```bash
pip install ansible # установка в системную директорию
pip install ansible --user # устнаовка в пользовательскую директорию
python -m venv env && . ./env/bin/activate && pip install ansible # установка в виртуальное окружение
```

### apt

В таком случае версия будет не самой последней.
Добавляем репозиторий. Для debian 11 нужен ubuntu focal.

```bash
echo 'deb https://ppa.launchpadcontent.net/ansible/ansible/ubuntu focal main' | sudo tee /etc/apt/sources.list.d/ansible.list
```

Нужно взять fingerprint с `https://launchpad.net/~ansible/+archive/ubuntu/ansible` или `apt update` и посмотреть на ошибку `NO_PUBKEY` и взять fingerprint для поиска. В данном случае это `93C4A3FD7BB9C367`

```bash
wget -O- "http://keyserver.ubuntu.com/pks/lookup?op=get&search=0x93C4A3FD7BB9C367" |
gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/ansible.gpg &>/dev/null
```

## Дополнительные зависимости

На хосте:
- `sshpass` если будет передаваться текстовый пароль

На клиенте:
- `python`

# Настройка

## ansible.cfg
Можно настроить конкретное окружение. Для этого создаём файл `ansible.cfg` в месте где будем запускать его.
- `host_key_checking` проверять fingerprint при новом подключении по ssh
- `inventory` указать какой использовать файл инвентаризации(клиентов) по умолчанию.

`ansible.cfg`
```
[defaults]
host_key_checking = false
inventory = hosts.txt
```

## hosts.txt

https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html

Файл инвентаризации в нём описываются клиенты к которым будет подключаться ansible для настройки



