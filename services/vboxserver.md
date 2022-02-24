---
title: vboxserver
description: 
published: true
date: 2022-02-24T12:55:06.417Z
tags: 
editor: markdown
dateCreated: 2022-02-24T12:55:06.417Z
---

# Virtualbox для сервера с web мордой.
https://anikin.pw/all/ustanovka-virtualbox-s-web-interfeysom-na-server/
OS: Debian 10 buster
Virtualbox: 6.1

# Установка
https://www.virtualbox.org/wiki/Linux_Downloads
Установка будет осуществлена.

```bash
echo "deb [arch=amd64] https://download.virtualbox.org/virtualbox/debian buster contrib" | sudo tee /etc/apt/sources.list.d/virtualbox.list
wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | sudo apt-key add -
sudo apt-get update
sudo apt-get install dkms virtualbox-6.1
```

Качаем extension pack с http://download.virtualbox.org/virtualbox/

```bash
wget http://download.virtualbox.org/virtualbox/6.1.14/Oracle_VM_VirtualBox_Extension_Pack-6.1.14.vbox-extpack
VBoxManage extpack install Oracle_VM_VirtualBox_Extension_Pack-6.1.14.vbox-extpack
```

# Настройка web соединения virtualbox
## Создадим юзера под которым будет работать веб сервер

```bash
adduser vbox
usermod -aG vboxusers vbox
```

Не забываем пароль, который мы назначили пользователю vbox. Он нам ещё потребуется.
## Настраиваем веб-сервис virtualbox

`/etc/vbox/vbox.cfg`

```bash
VBOXWEB_USER=vbox
VBOXWEB_HOST=127.0.0.1
VBOXWEB_PORT=18083

VBOXAUTOSTART_DB=/etc/vbox
VBOXAUTOSTART_CONFIG=/etc/vbox/vboxauto.cfg
```

## Установка phpvirtualbox
Для работы веб интерфейса нам потребуется веб-сервер(мы будем использовать nginx) и php. Установим их.

```bash
sudo apt install nginx php-fpm php-soap
```

Скачаем PHPVirtualBox с github https://github.com/phpvirtualbox/phpvirtualbox Будем использовать develop т.к. там есть наиболее полная реализация 6.1. Может быть уже есть реализация с версией, проверьте на github

```bash
wget https://github.com/phpvirtualbox/phpvirtualbox/archive/develop.zip
```

Поместим содержимое в папку `/var/www/phpvirtualbox`

Копируем пример конфига `cp /var/www/phpvirtualbox/config.php-example /var/www/phpvirtualbox/config.php`

Редактируем `/var/www/phpvirtualbox/config.php`

```php
var $username = 'vbox'; // ранее созданный пользователь
var $password = 'pass'; // его пароль
var $location = 'http://127.0.0.1:18083/'; // адрес настроенный в /etc/vbox/vbox.cfg переменно VBOXWEB_HOST
var $vrdeports = '9000-9100'; // диапазон портов для RDP соединений по умолчанию, который будет использоваться при создании новой виртуалки
var $vrdeaddress = '0.0.0.0'; // ip адресс для RDP по умолчанию, если не задать будет localhost
var $browserRestrictFolders = array('/home/vbox/'); // папка по умолчанию для выбора образов для установки.
```

Создаем файл виртуального хоста в nginx `/etc/nginx/sites-available/phpvirtualbox`

`fastcgi_pass unix:/var/run/php/php7.3-fpm.sock;` Это строка должна быть заполнена исходя из версии php-fpm
При запущенном сервисе `php7.3-fpm.service`, по пути `/var/run/php/php7.3-fpm.sock` будет устройство.

```
server {
        listen 8080;

        root /var/www/phpvirtualbox/;
        index index.php index.html index.htm;

        server_name _

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to index.html
                try_files $uri $uri/ /index.html;
        }


        # pass the PHP scripts to FastCGI server listening on 1$
        location ~ \.php$ {
                try_files $uri =404;
                fastcgi_split_path_info ^(.+.php)(.*)$;
                fastcgi_pass unix:/var/run/php/php7.3-fpm.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
        }

}
```

Ставим наш сайт в автозапуск `sudo ln -s /etc/nginx/sites-available/phpvirtualbox /etc/nginx/sites-enabled/phpvirtualbox`

Перезапускаем nginx и проверяем работу сервера, переходим по ip адресу и должно появится окно авторизации по умолчанию логин `admin:admin`

# Настройка автозапуска виртуалок из веб интерфейса
http://markov.site/2016/09/28/virtualbox/
в phpvirtualbox есть старая реализация автозапуска через vboxinit, включается переменной в `config.php` `startStopConfig`.
Но нам нужна стандартная реализация автозапуска через vboxautostart.service. Включается переменной `vboxAutostartConfig`, которую нужно добавить в `config.php`

в конфиг virtualbox `/etc/vbox/vbox.cfg` пишем

```
VBOXAUTOSTART_DB=/etc/vbox
VBOXAUTOSTART_CONFIG=/etc/vbox/vboxauto.cfg
```

А в конфиг автозапуска `/etc/vbox/vboxauto.cfg` прописать

```
default_policy = deny
vbox = {
    allow = true
    startup_delay = 5
}
```

Поменять права на папку `sudo chgrp vboxusers /etc/vbox` и `sudo chmod 1775 /etc/vbox`
От имени пользователя virtualbox web установить путь базы данных `su - vbox -c "VBoxManage setproperty autostartdbpath /etc/vbox"`
Теперь можно включать автозапуск в веб интерфейсе

## Включить автозапуск через консоль
Включение производится только от пользователя, которому принадлежит виртуалка

```
VBoxManage modifyvm «имя_виртуальной_машины» —autostart-enabled on
VBoxManage modifyvm «имя_виртуальной_машины» —autostop-type acpishutdown
```

autostop-type не работает, почему то этой команды больше нет, но API для web есть https://www.virtualbox.org/sdkref/_virtual_box_8idl.html#a03ad7f2af3ceb813a15cdc614db93c73