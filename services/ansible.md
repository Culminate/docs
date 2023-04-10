---
title: ansible
description: 
published: true
date: 2023-04-10T07:13:43.300Z
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
- `inventory` указать какой использовать inventory_file(файл хостов) по умолчанию.

`ansible.cfg`
```
[defaults]
host_key_checking = false
inventory = hosts.txt
```

## hosts

[Документация inventory_file](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html)
[Документация по ключевым словам](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#connecting-to-hosts-behavioral-inventory-parameters)

В файле инвентаризации описываются хосты к которым будет подключаться ansible для настройки. Имеет структуру ini файла, можно объединять хосты в группу и присваивать всей группе свойства

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

[Документация по Ad-Hoc](https://docs.ansible.com/ansible/latest/command_guide/intro_adhoc.html#intro-adhoc)
[Документация по встроенным командам](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/)

Можно использовать простые сценарии из командной строки.
Синтаксис `ansible -i <файл_хостов.txt> <группа> -m <модуль>`. Если мы задали в `ansible.cfg` переменную `inventory`, то файл хостов задавать не нужно. Если хотим запустить команду для всех хостов, то пишем all.

## Повышение привилегий



## Примеры разных команд

## Tabs {.tabset}
### ping
Соединение с хостом и получение ответа. Не имеет ничего общего с системной командой ping
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/ping_module.html

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

### setup
Получение данных о хосте
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/setup_module.html

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

### shell
Запуск оболочки для выполнения команд
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/shell_module.html

```
$ ansible all -m shell -a "uptime | cut -d',' -f1"
debian1 | CHANGED | rc=0 >>
 13:20:25 up 4 min
```

### copy
Копирование файла с сервера на хост
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html

```
$ ansible all -m copy -a "src=data.txt dest=~"
debian1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "checksum": "22596363b3de40b06f981fb85d82312e8c0ed511",
    "dest": "/home/debian/data.txt",
    "gid": 1000,
    "group": "debian",
    "md5sum": "6f5902ac237024bdd0c176cb93063dc4",
    "mode": "0644",
    "owner": "debian",
    "size": 12,
    "src": "/home/debian/.ansible/tmp/ansible-tmp-1681061269.8191762-558-146712978543920/source",
    "state": "file",
    "uid": 1000
}
```

### fetch
Копирование файла с хоста на сервер
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/fetch_module.html

По умолчанию создаёт директорию хоста и воссоздаёт дерево куда положит файл
```
$ ansible all -m fetch -a "src=/etc/hosts dest="
debian1 | CHANGED => {
    "changed": true,
    "checksum": "d5234cedd3184001eae464139064341046ba4afc",
    "dest": "/home/debian/user/ansible/debian1/etc/hosts",
    "md5sum": "9c8cc077c58b8b1fcfdee1050f4051bf",
    "remote_checksum": "d5234cedd3184001eae464139064341046ba4afc",
    "remote_md5sum": null
}
```

Если добавить `flat=yes`, то файл будет записан по названию переменной dest, но имейте в виду то что при множестве хостов файл будет перезаписан, так что нужно добавлять к ним идентификацию, например `inventory_hostname`
```
$ ansible all -m fetch -a "src=/etc/hosts dest=~/lapidus-{{ inventory_hostname }} flat=yes"
debian1 | CHANGED => {
    "changed": true,
    "checksum": "d5234cedd3184001eae464139064341046ba4afc",
    "dest": "/home/debian/lapidus-debian1",
    "md5sum": "9c8cc077c58b8b1fcfdee1050f4051bf",
    "remote_checksum": "d5234cedd3184001eae464139064341046ba4afc",
    "remote_md5sum": null
}
```

### file
Получение информации и управление файлами и директориями
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html

Получаем информацию о файле или директории
```
$ ansible all -m file -a "dest=~/data.txt"
debian1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "gid": 1000,
    "group": "debian",
    "mode": "0644",
    "owner": "debian",
    "path": "/home/debian/data.txt",
    "size": 12,
    "state": "file",
    "uid": 1000
}
```

Удаление файла или директории
```
$ ansible all -m file -a "dest=~/data.txt state=absent"
debian1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "path": "/home/debian/data.txt",
    "state": "absent"
}
```

### package
Менеджер пакетов, сам определяет на системе менеджер пакетов и устанавливает с помощью него пакет.
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/package_module.html

```
$ ansible all -m package -a "name=tree state=present" -b
debian1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1681107237,
    "cache_updated": false,
    "changed": true,
    "stderr": "",
    "stderr_lines": [],
    "stdout": "...",
    "stdout_lines": [
        "Reading package lists...",
        "Building dependency tree...",
        "Reading state information...",
        "The following NEW packages will be installed:",
        "  tree",
        "0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.",
        "Need to get 49.6 kB of archives.",
        ...
    ]
}
```

### reboot
Перезагружаем машину и ждём пока она снова появится в сети

```
$ ansible all -m reboot -b
debian1 | CHANGED => {
    "changed": true,
    "elapsed": 25,
    "rebooted": true
}
```

# Playbooks (Сценарии)
[Документация по ключевым словам](https://docs.ansible.com/ansible/latest/reference_appendices/playbooks_keywords.html)

# Links
[Youtube плейлист на уроки](https://www.youtube.com/playlist?list=PLg5SS_4L6LYufspdPupdynbMQTBnZd31N)