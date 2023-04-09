---
title: ansible
description: 
published: true
date: 2023-04-09T17:18:14.850Z
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

Параметры:
- `host_key_checking` проверять fingerprint при новом подключении по ssh
- `inventory` указать какой использовать файл инвентаризации(клиентов) по умолчанию.

`ansible.cfg`
```
[defaults]
host_key_checking = false
inventory = hosts.txt
```

## hosts

https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html

Файл инвентаризации в нём описываются хосты к которым будет подключаться ansible для настройки

Имеет структуру ini файла, можно объединять хосты в группу и присваивать всей группе свойства

## Tabs {.tabset}
### Простой пример
```
debian_host1 ansible_host=192.168.22.1 ansible_port=2222 ansible_user=root ansible_password=MYPASS1234
```

### Группа с общими настройками
```
[my_hosts]
debian_host1 ansible_host=192.168.22.1
debian_host2 ansible_host=192.168.22.2

[my_hosts:vars]
ansible_port=2222 ansible_user=root ansible_password=MYPASS1234
```

### Использование диапазонов
```
[my_hosts]
debian_host1 ansible_host=192.168.22.[1:2]

[my_hosts:vars]
ansible_port=2222 ansible_user=root ansible_password=MYPASS1234
```

### Подгруппы
```
[group1]
10.10.0.1

[group2]
10.10.0.2

[group3]
10.10.0.3

[groupall:children]
group1
group2
group3
```

# Ad-Hoc команды

Можно использовать простые сценарии из командной строки.
Синтаксис `ansible -i <файл_хостов.txt> <группа> -m <модуль>`. Если мы задали в `ansible.cfg` переменную `inventory`, то файл хостов задавать не нужно. Если хотим запустить команду для всех хостов то пишем all.

Примеры разных команд:

# {.tabset}
## ping - соединение с хостом
```
$ ansible all -m ping
debian1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

## setup - получение данных о хосте

```
$ ansible all -m setup
debian1 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "192.168.50.201"
        ],
        ...
        Большой список из параметров, которые можно использовать в сценариях
        ...
}
```

## shell

```
$ ansible all -m shell -a "uptime"
debian1 | CHANGED | rc=0 >>
 13:15:48 up 0 min,  1 user,  load average: 0.00, 0.00, 0.00
```