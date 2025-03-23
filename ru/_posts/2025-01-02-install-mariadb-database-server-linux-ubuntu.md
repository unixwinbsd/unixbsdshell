---
title: Как установить сервер базы данных MariaDB на Linux Ubuntu
date: "2025-01-02 13:00:09 +0500"
id: install-mariadb-database-server-linux-ubuntu
lang: ru
layout: single
author_profile: true
categories:
  - Linux
tags: "DataBase"
excerpt: "Способ использования MariaDB почти такой же, как и у сервера MySQL, поскольку оба они происходят из одного и того же чрева."
keywords: mariadb, database, server, linux, ubuntu, client, mysql, mysql server, freebsd
---

База данных — залог успеха любого бизнеса. Базы данных являются основой всех приложений, будь то мобильные, веб-приложения или приложения Интернета вещей. Независимо от того, пользуетесь ли вы телефоном, получаете рецепт или совершаете финансовую транзакцию, за всем этим вы обнаружите базу данных MariaDB. Имея более 1 миллиарда загрузок, почти 200 000 вкладов с открытым исходным кодом и охватывая более 1 миллиарда пользователей через дистрибутивы Linux, MariaDB не просто обслуживает рынок реляционных баз данных, мы помогаем формировать его будущее.

MariaDB — это база данных. MariaDB очень похожа на MySQL (систему управления базами данных) и, по сути, является ответвлением MySQL. Базы данных MariaDB используются для различных целей, таких как хранение данных, электронная коммерция, функции корпоративного уровня и приложения для ведения записей.

MariaDB позволит вам эффективно выполнять все ваши рабочие нагрузки; он работает с любой облачной базой данных и работает в любом масштабе, как малом, так и большом. MariaDB — это ответвление реляционной системы управления базами данных MySQL, разработанное сообществом и предназначенное для свободного распространения под лицензией GNU GPL.

Будучи ответвлением ведущей системы программного обеспечения с открытым исходным кодом, MariaDB примечательна тем, что ее возглавляли первоначальные разработчики MySQL, которые создали ответвление из-за опасений по поводу ее приобретения корпорацией Oracle.
<br><br/>
## 1. Установка программного обеспечения MariaDB
По умолчанию пакеты MariaDB недоступны в репозитории Ubuntu, вам придется получить последнюю стабильную версию непосредственно с официального сайта проекта MariaDB или Github. Чтобы установить MariaDB в системе Linux, такой как Ubuntu, Debian или другие, вы можете следовать приведенному ниже сценарию.

```
$ curl --silent https://supplychain.mariadb.com/MariaDB-Server-GPG-KEY \
    | gpg --dearmor | \
    sudo tee /etc/apt/keyrings/MariaDB-Server.gpg > /dev/null

$ curl --silent https://supplychain.mariadb.com/MariaDB-MaxScale-GPG-KEY \
    | gpg --dearmor | \
    sudo tee /etc/apt/keyrings/MariaDB-MaxScale.gpg > /dev/null

$ curl --silent https://supplychain.mariadb.com/MariaDB-Enterprise-GPG-KEY \
    | gpg --dearmor | \
    sudo tee /etc/apt/keyrings/MariaDB-Enterprise.gpg > /dev/null
```
После установки GPG-KEY откройте файл MariaDB в папке /etc/apt/sources.list.d/mariadb.sources.

```
# MariaDB Server
# To use a different major version of the server, or to pin to a specific minor
# version, change URI below.
Types: deb
URIs: https://dlm.mariadb.com/repo/mariadb-server/10.11/repo/ubuntu
Suites: jammy
Components: main
Architectures: amd64 arm64
Signed-By: /etc/apt/keyrings/MariaDB-Server.gpg

# MariaDB MaxScale
# To use the latest stable release of MaxScale, use "latest" as the version
# To use the latest beta (or stable if no current beta) release of MaxScale, use
# "beta" as the version
Types: deb
URIs: https://dlm.mariadb.com/repo/maxscale/latest/apt
Suites: jammy
Components: main
Architectures: amd64 arm64
Signed-By: /etc/apt/keyrings/MariaDB-MaxScale.gpg

# MariaDB Tools
# MariaDB Tools are a collection of advanced command-line tools.
Types: deb
URIs: http://downloads.mariadb.com/Tools/ubuntu
Suites: jammy
Components: main
Architectures: amd64
Signed-By: /etc/apt/keyrings/MariaDB-Enterprise.gpg

# -*- mode: debsources; indent-tabs-mode: nil; tab-width: 4; -*-
```

После этого создайте файл настроек, чтобы задать пакетам из репозитория MariaDB наивысший приоритет, чтобы избежать конфликтов с пакетами из ОС и других репозиториев:

```
Package: *
Pin: origin dlm.mariadb.com
Pin-Priority: 1000
```

После этого запустите процесс обновления.

```
$ sudo apt update
```

После добавления ключа и репозитория, а также обновления базы данных пакетов вы можете напрямую установить MariaDB с помощью следующей команды.

```
$ sudo apt install mariadb-server
```

Примечания:
Если сервер Oracle MySQL уже установлен, он будет удален, но не будет удален без подтверждения. Файлы конфигурации сервера MySQL будут храниться и использоваться MariaDB.

В системах Ubuntu MariaDB работает как служба Systemd с именем mariadb.service. Службу нельзя запустить напрямую, необходимо дать команду на запуск MariaDB.

```
$ systemctl status mariadb.service
? mariadb.service - MariaDB 10.10.2 database server
    Loaded: loaded (/lib/systemd/system/mariadb.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/mariadb.service.d
            +-migrated-from-my.cnf-settings.conf
    Active: inactive (dead)
    Docs: man:mariadbd(8)
            https://mariadb.com/kb/en/library/systemd/
```
<br><br/>
## 2. Конфигурация MariaDB
Представленная здесь конфигурация сильно отличается от той, которую программные пакеты обычно устанавливают по умолчанию. Это подготовка к задачам, которые должен выполнить сервер.

В системах Ubuntu MariaDB запускается и управляется systemd как служебная единица mariadb.service. Для запуска, остановки или перезапуска сервера MariaDB можно использовать команду systemctl.

```
$ systemctl start mariadb.service
$ systemctl restart mariadb.service
$ systemctl stop mariadb.service
```

Команда «reload» не поддерживается в сервисе MariaDB. Вы можете просматривать сообщения о состоянии и ошибках, записанные в журнале systemd, а для мониторинга сервера можно использовать команду journalctl.

```
$ journalctl -u mariadb.service
$ journalctl -f -u mariadb.service
```

### a. Настройка системы MariaDB

Файл модуля службы MariaDB устанавливается в папку /lib/systemd/system/mariab.service. Для нормальной работы MariaDB необходимо применить пользовательские параметры. Для этого нужно изменить файл /lib/systemd/system/mariab.service.

```
$ sudo edit mariab.service
```
Пример скрипта systemd вы можете увидеть ниже.

```
# etc/systemd/system/mariadb.service.d/override.conf
[Unit]
After=sys-devices-virtual-net-wg0.device unbound.service
BindsTo=sys-devices-virtual-net-wg0.device
```

### b. mysqld_safe

До версии MariaDB 10.1.8 сервер запускался скриптом инициализации или как служба upstart в большинстве систем UNIX. Этот скрипт также применяет параметры, найденные в файле конфигурации MySQL в разделе [mysqld_safe] или [safe_mysqld].

Примечания:
При редактировании файла mysqld_safe принято, что имена переменных всегда указываются с символом «_», а параметры — с символом «-».

Всегда используйте «?» (дефис) при настройке параметров в файлах конфигурации или параметрах командной строки. Например, key-buffer-size = 64K.

Всегда используйте «_» (подчеркивание) в SQL-запросах на сервере. Например, ПОКАЗАТЬ ПЕРЕМЕННЫЕ, КАК 'key_buffer_size';

Основной файл конфигурации MariaDB называется /etc/mysql/mariadb.cnf, при необходимости вы можете изменить этот файл. Ниже приведен пример скрипта файла /etc/mysql/mariadb.cnf, который мы настроили для используемой нами системы.

```
#
# MariaDB database server version 10.10.2 configuration file.
#
# For explanations see:
#  * https://roll.urown.net/server/mariadb/
#  * https://mariadb.com/kb/en/library/server-system-variables/
#


[client]
#
# Character-Set
# Default: Latin1
default-character-set = utf8mb4

#
# UNIX Sockets & TCP/IP
port = 3306
socket = /run/mysqld/mysqld.sock


[mysqld]
#
# Basic Settings
#
user = mysql
pid-file = /run/mysqld/mysqld.pid
basedir = /usr
datadir = /var/lib/mysql
tmpdir = /tmp
lc_messages_dir = /usr/share/mysql
lc_messages = en_US
#
# If applications support it, this stricter sql_mode prevents some
# mistakes like inserting invalid dates etc.
#sql_mode       = NO_ENGINE_SUBSTITUTION,TRADITIONAL

# The default storage engine. The default storage engine must be enabled at
# server startup or the server won't start.
# Default: InnoDB
default_storage_engine = InnoDB

#
# Character-Set
# Default: Latin1
character-set-server = utf8mb4
collation-server = utf8mb4_general_ci

#
# UNIX Sockets & TCP/IP
#
socket = /run/mysqld/mysqld.sock

# By default, the MariaDB server listens for TCP/IP connections on a network
# socket bound to a single address, 0.0.0.0. You can specify an alternative when
# the server starts using this option; either a host name, an IPv4 or an IPv6
# address. In Debian and Ubuntu, the default bind_address is 127.0.0.1, which
# binds the server to listen on localhost only.
# Debian-Default: 127.0.0.1
# Default: 0.0.0.0 / :: (All available IPv4 and IPv6 interfaces)
bind-address = ::
port = 3306

# If set to ON, only IP addresses are used for connections. Host names are not
# resolved. All host values in the GRANT tables must be IP addresses (or
# localhost).
# Default: OFF
skip-name-resolve = ON

# Set longer periods to avoid timeout errors
connect_timeout = 600
wait_timeout = 28800


# This was formally known as [safe_mysqld]. Both versions are currently parsed.
[mysqld_safe]
socket = /run/mysqld/mysqld.sock
nice = 0


[mysqldump]
quick
quote-names
max_allowed_packet = 16M


[mysql]
#no-auto-rehash # faster start of mysql but no tab completion


[isamchk]
key_buffer = 16M

#
# Additional settings will override anything in this file!
# The files must end with '.cnf', otherwise they'll be ignored.
#
!includedir /etc/mysql/conf.d/

# -*- mode: ini; tab-width: 4; indent-tabs-mode: nil -*-
```

Важные настройки в скрипте выше следующие:

- skip-name-resolve = ON, поскольку наш DNS-сервер будет получать данные из своей базы данных.
- bind-address, настроен на прослушивание всех интерфейсов. Потому что нам нужен сетевой доступ для репликации с другими серверами. К сожалению, вы можете настроить только один интерфейс (обычно localhost) или все из них. Поскольку нам нужен локальный хост И внешний доступ, а не все наши клиенты могут использовать сокеты UNIX, нам нужно открыть их все. Это означает, что нам необходимо очень тщательно управлять правами доступа пользователей к серверу базы данных, а также нам необходим брандмауэр для блокировки нежелательного удаленного доступа.

В приведенном выше примере сценария мы не изменяли файлы debian.cnf и debian-start. Вы также можете создать дополнительные файлы из /etc/mysql/conf.d/ и /etc/mysql/mariadb.conf.d/, цель которых — обеспечить совместимость MariaDB с сервером Oracle MySQL.

Параметры конфигурации, совместимые с обоими продуктами, следует хранить в /etc/mysql/conf.d/, а параметры, работающие только с MariaDB, следует хранить в /etc/mysql/mariadb.conf.d/. Файл /etc/mysql/conf.d/mariadb.cnf загрузит его, но только для продуктов, совместимых с MariaDB.

### c. Character Sets

Чтобы использовать наборы символов, всегда следует использовать UTF-8 вместо Latin1 в качестве кодировки по умолчанию как для сервера, так и для клиента. Затем откройте /etc/mysql/my.cnf и раскомментируйте три строки UTF-8. Как в примере ниже.

```
[client]
#
# Character-Set
# Default: Latin1
default-character-set = utf8mb4

[mysqld]
#
# Character-Set
# Default: Latin1
character-set-server = utf8mb4
collation-server = utf8mb4_general_ci
```

После завершения настройки перезапустите сервер MariaDB с помощью следующей команды.

```
$ sudo systemctl restart mariadb.service
```

### d. UNIX Sockets & TCP/IP

Текущие версии серверов MariaDB и MySQL можно настроить на прослушивание только одного IP-адреса, одного IP-адреса или всех IP-адресов, независимо от версии IP.

Таким образом, для хоста с двойным вложением, который прослушивает как IPv6, так и IPv4, или хоста, который должен быть доступен как через интерфейс localhost/127.0.0.1, так и через реальный сетевой интерфейс, сервер должен быть настроен на прослушивание всех интерфейсов. Сервер не может быть ограничен, например, прослушиванием локального хоста и одного IP-адреса. Сервер может прослушивать один или все из них.

Отредактируйте соответствующие разделы файла /etc/mysql/my.cnf, как показано в примере ниже.

```
[client]

#
# UNIX Sockets & TCP/IP
port = 3306
socket = /run/mysqld/mysqld.sock

[mysqld]
# UNIX Sockets & TCP/IP
#
socket = /run/mysqld/mysqld.sock

# By default, the MariaDB server listens for TCP/IP connections on a network
# socket bound to a single address, 0.0.0.0. You can specify an alternative when
# the server starts using this option; either a host name, an IPv4 or an IPv6
# address. In Debian and Ubuntu, the default bind_address is 127.0.0.1, which
# binds the server to listen on localhost only.
# Debian-Default: 127.0.0.1
# Default: 0.0.0.0 / :: (All available IPv4 and IPv6 interfaces)
bind-address = ::
port = 3306

# If set to ON, only IP addresses are used for connections. Host names are not
# resolved. All host values in the GRANT tables must be IP addresses (or
# localhost).
# Default: OFF
skip-name-resolve = ON

# Set longer periods to avoid timeout errors
connect_timeout = 600
wait_timeout = 28800
```

Способ использования MariaDB почти такой же, как и у сервера MySQL, поскольку оба они происходят из одного и того же чрева. MariaDB также обладает мощными функциями, гибкостью, и все ее возможности практически такие же, как у сервера MySQL. Независимо от того, являетесь ли вы администратором баз данных, разработчиком или просто интересуетесь базами данных с открытым исходным кодом, MariaDB предлагает привлекательный вариант с разработкой, осуществляемой сообществом, высокой производительностью и широким спектром вариантов использования.
