---
title: ansible
description: 
published: true
date: 2023-04-09T12:12:09.867Z
tags: 
editor: markdown
dateCreated: 2023-04-09T12:11:06.065Z
---

# Ansible
Средство автоматизации множества хостов по ssh.

OS: Debian 11

## Установка
Работает только на linux, несмотря на то что это python.

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
