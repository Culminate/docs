# apt-cache-ng кэширование apt(.deb) пакетов

## Использование при установке netinst образа debian

При настройке внешних репозиториев настроить доступ к ним через прокси ввести адрес apt-cache-ng прокси Например: `http://192.168.1.15:3142`

После установки пакетов не забыть удалить настройку прокси `Acquire::http::proxy` находящуюся в `/etc/apt/apt.conf` и перенастроить прокси согласно пункту [Настройка на клиентах](#_2)

## Настройка на сервере

Файл настроек
/etc/apt-cacher-ng/acng.conf

```bash
CacheDir: /drive/apt-cacher-ng # расположение кэша пакетов

# Количество дней до удаления невостребованного пакета
ExThreshold: 90

# Объем доступного кэша в мегабайтах
ExStartTradeOff: 51200m

# Чтобы резопзитории не двоились согласно имени хоста, то нужно задать соответвие зеркал и названия папки.
# Все скачанные пакеты с <входящие адреса репозиториев> будут помещены <имя папки в папке кэша>
# <option> уточнение папки для кажого зеркала
# Целевой репозиторий, даже если пакета нет, то пакет будет качаться не с входящего адреса а с <целевые репозитории>
# Remap-<имя папки в папке кэша>: <входящие адреса репозиториев> <option>; <целевые репозитории>;
Remap-debrep: file:deb_mirror*.gz /debian ; file:backends_debian # Debian Archives
```

## Настройка на клиентах

Если задавать proxy через `Acquire::http::proxy`, то при неработоспособности сервера apt-cache-ng, apt не сможет загружать пакеты напрямую.
Поэтому сделаем его через скрипт определения прокси.

создать файл /etc/apt/Proxy-Auto-Detect c правами 755 и владельцем root

```bash
touch /etc/apt/Proxy-Auto-Detect
chown root:root /etc/apt/Proxy-Auto-Detect
chmod 755 /etc/apt/Proxy-Auto-Detect

```

И наполнением

```bash
#!/bin/bash

# WARNING: only for apt-cache-ng proxy

show_proxy_messages=0

# proxies for checking
try_proxies=(
192.168.1.15:3142
)

print_msg() {
    # \x0d clears the line so [Working] is hidden
    [ "$show_proxy_messages" = 1 ] && printf '\x0d%s\n' "$1" >&2
}

for proxy in "${try_proxies[@]}"; do
    # get info web page
    curl -sSf http://$proxy &> /dev/null;
    if [ $? -eq 22 ]; then
        proxy=http://$proxy
        print_msg "Proxy that will be used: $proxy"
        echo "$proxy"
        exit
    fi
done
print_msg "No proxy will be used"

# if proxy not available
echo DIRECT
```

создать файл /etc/apt/apt.conf.d/30Proxy-Auto-Detect с правами 644 и владельцем root

```bash
touch /etc/apt/apt.conf.d/30Proxy-Auto-Detect
chown root:root /etc/apt/Proxy-Auto-Detect
chmod 644 /etc/apt/Proxy-Auto-Detect
```

И наполнением

```bash
Acquire::http::Proxy-Auto-Detect /etc/apt/Proxy-Auto-Detect;
```