---
title: Как подключить FreeBSD к Windows с помощью Samba
date: "2024-01-26 19:00:00 +0300"
id: samba-on-freebsd-connect-windows
lang: ru
layout: single
author_profile: true
categories:
  - FreeBSD
excerpt: "С помощью Samba всю вашу работу можно легко выполнить в одной сети."
tags: "SysAdmin"
keywords: samba, freebsd, windows, openbsd, configuration, setup, server, transfer, client
---

У вас есть несколько устройств в офисе, компании, дома, школе или где-то еще? Как вы передаете данные между компьютерами? Как вы передаете данные между компьютерами? Используете ли вы флэш-накопители или SD-карты для передачи данных между компьютерами? Этот вопрос возникает, когда вы работаете с несколькими компьютерами в одном месте.

Samba — это ответ, с Samba вся ваша работа может быть легко выполнена в одной сети. Вам не нужно тратить драгоценное время на быструю и легкую передачу файлов, документов и других больших элементов по локальной сети. Это одноразовая настройка, а затем несколькими щелчками мыши вы можете обмениваться файлами между компьютерами.

Если вы обычно работаете с Windows и хотите обмениваться файлами или документами по локальной сети или просто хотите, чтобы файлы или документы были доступны всем компьютерам, FreeBSD и Samba — правильный выбор. Вы можете использовать сервер FreeBSD в качестве шлюза, чтобы к файлам или документам могли обращаться несколько компьютеров.

На рисунке ниже показан процесс соединения файлов или документов между FreeBSD (samba) и Windows.

![connecting files or documents between FreeBSD and samba on Windows](/img/connecting files or documents between FreeBSD and samba on Windows.jpg)

Каждый пользователь, желающий получить доступ к файлу или документу, должен войти в систему и воспользоваться полученными правами доступа. Успешно вошедшие в систему пользователи могут читать, создавать или изменять файлы на компьютере, на котором они работают.

Доступ к серверу Samba также может осуществляться без пароля, поэтому к нему может получить доступ каждый. Хоть это и не рекомендуется, но некоторым людям это действительно необходимо. Например, если вам нужно очень быстро открыть доступ и нет времени на настройку ограниченного доступа. Настоятельно рекомендуется, чтобы каждый пользователь использовал пароль для входа в систему. Это необходимо для защиты данных или документов, чтобы никто не мог их открыть.

# Спецификации сервера Samba
- ОС: FreeBSD 13.2
- Имя хоста: ns3
- IP-адрес: 192.168.5.2
- интерфейсы = nfe0
- Версия Samba: samba416
- Версия Python: python39
- Зависимости: p5-Parse-Yapp, libiconv, curl, py-wsdd

## А. Подготовка к установке
Прежде чем приступить к установке Samba, необходимо установить зависимости, необходимые Samba. Цель — обеспечить нормальную работу сервера Samba. Вам необходимо установить несколько зависимостей, но наиболее важными из них являются Python39, p5-Parse-Yapp, libiconv, py-wsdd. Остальные зависимости обычно устанавливаются автоматически.

```
root@ns3:~ # pkg install python39 p5-Parse-Yapp libiconv net/py-wsdd
```

После установки вышеуказанных зависимостей приступайте к установке Samba. В этой статье мы будем использовать систему портов для установки Samba.

```
root@ns3:~ # cd /usr/ports/net/samba416
root@ns3:/usr/ports/net/samba416 # make config
```

При запуске команды «make config» появится меню параметров Samba. Необходимо отметить опцию «PYTHON3».

Продолжайте выполнять команду «make install clean».

```
root@ns3:/usr/ports/net/samba416 # make install clean
```

Создайте сценарий Start Up rc.d, чтобы сервер Samba мог запускаться автоматически. Откройте файл «/etc/rc.conf» и введите приведенный ниже скрипт в файл «/etc/rc.conf».

```
samba_server_enable="YES"
nmbd_enable="YES"
smbd_enable="YES"
winbindd_enable="YES"
samba_server_config="/usr/local/etc/smb4.conf"
wsdd_enable="YES"
```

В конце установки порт net/samba416 не включает файл конфигурации «smb4.conf», необходимый для запуска Samba, поэтому мы создадим файл «smb4.conf». Чтобы упростить написание скрипта файла /m, мы создадим скрипт из двух частей.

- Глобальные настройки
- Определение обмена.

Вот пример полного скрипта "/usr/local/etc/smb4.conf". Настройте его в соответствии с техническими характеристиками вашего сервера Samba.

```
#======================= Global Settings =====================================
[global]

workgroup = WORKGROUP
server string = FreeBSD %v (%h)
server role = standalone server
hosts allow = 192.168.5. 127.
;  guest account = pcguest
log file = /usr/local/samba/var/log.%m
log level = 1
max log size = 50
;  realm = MY_REALM
passdb backend = tdbsam
bind interfaces only = yes
#interfaces = nfe0 lo0
interfaces = 192.168.5.0/24 127.0.0.1
wins support = no
wins server = 192.168.5.2
local master = yes
preferred master = yes

#============================ Share Definitions ==============================
[Personal]
   comment = Home Folder
   browseable = yes
   writeable = yes
   valid users = %U
   create mode = 0644
   directory mode = 0755
   path = /usr/local/etc/sambashare/personal
   guest ok = no

[Public]
   comment = Public Folder
   path = /usr/local/etc/sambashare/public
   browseable = yes
   writeable = yes
   guest ok = no
   create mode = 0664
   directory mode = 0775
   valid users = %U

[Shared] 
   comment = Shared Folder
   path = /usr/local/etc/sambashare/shared
   browseable = yes
   writeable = yes
   guest ok = no
   create mode = 0666
   directory mode = 0777
   valid users = %U

[Guest] 
   comment = Guest Folder
   path = /usr/local/etc/sambashare/guest
   browseable = yes
   writeable = yes
   guest ok = no
   create mode = 0666
   directory mode = 0777
   valid users = %U
```

## Б. Подготовка конфигурации
Этот процесс является неотъемлемой частью процесса установки. Первый шаг, который нам необходимо сделать после создания файла конфигурации «smb4.conf», — это создать каталог.

```
root@ns3:~ # mkdir -p /usr/local/etc/sambashare/personal
root@ns3:~ # mkdir -p /usr/local/etc/sambashare/public
root@ns3:~ # mkdir -p /usr/local/etc/sambashare/shared
root@ns3:~ # mkdir -p /usr/local/etc/sambashare/guest
```

После этого выполните команду «chmod», эта команда используется для предоставления прав доступа/разрешений владельцу, обычным пользователям и непользователям. Следуем сценарию из файла «smb4.conf».

```
root@ns3:~ # chmod 775 /usr/local/etc/sambashare/personal
root@ns3:~ # chmod 775 /usr/local/etc/sambashare/public
root@ns3:~ # chmod 777 /usr/local/etc/sambashare/shared
root@ns3:~ # chmod 777 /usr/local/etc/sambashare/guest
```

Следующим шагом будет редактирование файла «/boot/loader.conf» и вставка в него скрипта, приведённого ниже.

```
smbfs_load="YES"
```

Для защиты общих данных или документов мы создаем пользователя и пароль. Имея имя пользователя и пароль, не каждый может получить доступ к файлу или документу.

```
root@ns3:~ # pw add group samba
root@ns3:~ # pw add group freebsd
```

Приведенный выше скрипт используется для создания группы с именем «samba». После создания группы приступайте к созданию пользователей. В этом примере мы создадим 3 пользователей: Джона, Мэри и Питера, которые будут в группе Samba.

```
root@ns3:~ # pw add user -n john -g samba -s /sbin/nologin -c "samba john"
root@ns3:~ # pw add user -n mary -g samba -s /sbin/nologin -c "samba mary"
root@ns3:~ # pw add user -n peter -g samba -s /sbin/nologin -c "samba peter"
root@ns3:~ # pw add user -n jackson -g freebsd -s /sbin/nologin -c "samba guest"
```

Укажите пароль для каждого пользователя.

```
root@ns3:~ # pdbedit -a -u john -f "john chaplin"
root@ns3:~ # pdbedit -a -u mary -f "mary rose"
root@ns3:~ # pdbedit -a -u peter -f "peter joe"
root@ns3:~ # pdbedit -a -u jackson -f "jackson samba"
```

Чтобы вам было легче, посмотрите на картинку ниже.

![create samba user and group in freebsd](/img/create samba user and group in freebsd.jpg)

Смущенный? Теперь продолжим с командой chown. Эта команда используется для смены владельца файла или каталога.

```
root@ns3:~ # chown -R john:samba /usr/local/etc/sambashare/shared
root@ns3:~ # chown -R mary:samba /usr/local/etc/sambashare/public
root@ns3:~ # chown -R peter:samba /usr/local/etc/sambashare/personal
root@ns3:~ # chown -R jackson:freebsd /usr/local/etc/sambashare/guest
```

Запустите сервер Samba.

```
root@ns3:~ # service samba_server restart
```

## C. Конфигурация Windows
На этом этапе мы начнем настройку в системной среде Windows. Откройте «C:\Windows\System32\drivers\etc», после чего введите скрипт, приведенный ниже.

```
192.168.5.2    ns3
```

В приложении Samba, которое мы изучаем, FreeBSD является сервером Samba, а Windows — клиентом Samba. Поэтому все файлы или документы, которыми вы хотите поделиться, должны быть размещены на сервере FreeBSD. Любой пользователь, желающий использовать файл или документ, может получить к нему доступ только через Windows. В крупных средах, таких как школы, больницы или компании, необходим сетевой администратор для управления правами доступа к файлам или документам, которые являются общими или доступными для пользователей.

Вы копируете все файлы или документы на сервер FreeBSD в соответствии с правами доступа пользователя. После этого запустите «chmod», как в примере ниже.

```
root@ns3:~ # chmod 644 /usr/local/etc/sambashare/shared/*
root@ns3:~ # chmod 644 /usr/local/etc/sambashare/public/*
root@ns3:~ # chmod 666 /usr/local/etc/sambashare/personal/*
root@ns3:~ # chmod 666 /usr/local/etc/sambashare/guest/*
```

Выполняйте указанную выше команду каждый раз, когда файл или документ добавляется на сервер FreeBSD.

Любой пользователь, желающий получить доступ к файлу или документу, может открыть его через проводник Windows. Хорошо, давайте начнем испытывать приложение Samba на клиенте Windows. Откройте проводник Windows и введите «\\192.168.5.2\personal» или «\\ns3\personal».

Чтобы удалить подключение к серверу Samba в Windows, запустите «cmd» и введите команду «net use * /delete».

Вы также можете попробовать выполнить команду ниже в оболочке «cmd».

```
C:\Users\blogspot>net use \delete \\192.168.5.2\personal
C:\Users\blogspot>net use \delete \\192.168.5.2\public
C:\Users\blogspot>net use \delete \\ns3
```

Samba имеет множество функций и применений. Итак, вам придется многое попробовать на своем сервере FreeBSD, а также прочитать множество статей с инструкциями по Samba в поисковой системе Google. Обсуждение в этой статье — лишь часть изучения самбы. Однако процесс установки и настройки практически такой же, если вы хотите объединить его с сервером DHCP или DNS Bind.




