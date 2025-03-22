---
title: Как установить Ubuntu на FreeBSD | Совместимость двоичных файлов Linux
date: "2024-03-16 14:15:00 +0300"
id: freebsd-install-ubuntu-linux-binary
lang: ru
layout: single
categories:
  - Linux
excerpt: "Встроенная в FreeBSD двоичная совместимость Linux позволяет пользователям FreeBSD, работающим на 32-битной архитектуре i386, 64-битной архитектуре amd64 или 64-битной архитектуре arm64"
tags: "ubuntu, freebsd, linux, openbsd, configuration, setup, apt, commerce, binary"
keywords: ubuntu, freebsd, linux, openbsd, configuration, setup, apt, commerce, binary
---

В этой статье обсуждается процедура установки базовой системы Ubuntu во встроенную в FreeBSD бинарную совместимость Linux, чтобы приложения для рабочего стола на базе Ubuntu и Debian, такие как Signal, Spotify и Netflix, могли работать непосредственно на FreeBSD. В обсуждении в этой статье используется Ubuntu 20.04, которая будет установлена ​​в систему FreeBSD 13.2 Stable для 64-битной архитектуры AMD64.

Встроенная в FreeBSD бинарная совместимость Linux позволяет пользователям FreeBSD, работающим на 32-битной архитектуре i386, 64-битной архитектуре amd64 или 64-битной архитектуре arm64, устанавливать и запускать 32-битные и 64-битные двоичные файлы Linux непосредственно на машинах FreeBSD. Это делается с помощью таблиц системных вызовов, что означает, что приложения Linux могут работать без эмуляции или виртуализации. Встроенная в FreeBSD бинарная совместимость Linux была представлена ​​в 90-х годах.

## Ubuntu Linux Kernel
Первый шаг, который необходимо сделать для запуска Ubuntu на FreeBSD, — это активировать ядро ​​Ubuntu Linux на машине FreeBSD. По сути, FreeBSD имеет встроенную бинарную совместимость с Linux. Это позволяет запускать двоичные файлы Ubuntu и Linux. Это активируется путем загрузки модуля ядра Linux. Ниже приведен kenel Linux во FreeBSD.

```
kldload linux
kldload linux64
kldload fdescfs
kldload linprocfs
kldload tmpfs
kldload linsysfs
```

Чтобы ядро ​​Linux могло быть активным на машине FreeBSD, чтобы при перезагрузке/перезапуске компьютера ядро ​​могло быть напрямую прочитано FreeBSD. Введите скрипт ядра Linux в файл /boot/loader.conf.

```
root@ns1:~ # ee /boot/loader.conf
linux_load="YES"
linux64_load="YES"      
fdescfs_load="YES"
linprocfs_load="YES"
tmpfs_load="YES"
linsysfs_load="YES"
```

Скрипт выше сделает ядро ​​Ubuntu Linux постоянным в системе FreeBSD. После того, как ядро ​​станет постоянным, создайте скрипт rc.d в файле /etc/rc.conf.

```
root@ns1:~ # ee /etc/rc.conf
kld_list="linux linux64 cuse fusefs /boot/modules/i915kms.ko"
linux_enable="YES"
ubuntu_enable="YES"
```

## Создайте скрипт базовой системы Ubuntu rc.d
Введите следующий скрипт, чтобы начать активацию базовой системы Ubuntu.

```
root@ns1:~ # touch /usr/local/etc/rc.d/ubuntu
root@ns1:~ # chmod +x /usr/local/etc/rc.d/ubuntu
root@ns1:~ # ee /usr/local/etc/rc.d/ubuntu
#!/bin/sh
#
# PROVIDE: ubuntu
# REQUIRE: archdep mountlate
# KEYWORD: nojail
#

. /etc/rc.subr

name="ubuntu"
desc="Ubuntu for FreeBSD Linux Binary Compatibility"
rcvar="ubuntu_enable"
start_cmd="${name}_start"
stop_cmd=":"

unmounted()
{
  [ `stat -f "%d" "$1"` == `stat -f "%d" "$1/.."` -a `stat -f "%i" "$1"` != `stat -f "%i" "$1/.."` ]
}

ubuntu_start()
{
  local _tmpdir
  load_kld -e 'linux(aout|elf)' linux
  case `sysctl -n hw.machine_arch` in
    amd64)
      load_kld -e 'linux64elf' linux64
      ;;
  esac
  if [ -x "/compat/ubuntu/sbin/ldconfigDisabled" ]; then
    _tmpdir=`mktemp -d -t linux-ldconfig`
    /compat/ubuntu/sbin/ldconfig -C ${_tmpdir}/ld.so.cache
    if ! cmp -s "${_tmpdir}/ld.so.cache" "/compat/ubuntu/etc/ld.so.cache"; then
      cat "${_tmpdir}/ld.so.cache" > "/compat/ubuntu/etc/ld.so.cache"
    fi
    rm -rf ${_tmpdir}
  fi
  load_kld pty
  if [ `sysctl -ni kern.elf64.fallback_brand` -eq "-1" ]; then
    sysctl kern.elf64.fallback_brand=3 > /dev/null
  fi
  if [ `sysctl -ni kern.elf32.fallback_brand` -eq "-1" ]; then
    sysctl kern.elf32.fallback_brand=3 > /dev/null
  fi
  sysctl compat.linux.emul_path="/compat/ubuntu"
  unmounted "/compat/ubuntu/dev" && (mount -o nocover -t devfs devfs "/compat/ubuntu/dev" || exit 1)
  unmounted "/compat/ubuntu/dev/fd" && (mount -o nocover,linrdlnk -t fdescfs fdescfs "/compat/ubuntu/dev/fd" || exit 1)
  unmounted "/compat/ubuntu/dev/shm" && (mount -o nocover,mode=1777 -t tmpfs tmpfs "/compat/ubuntu/dev/shm" || exit 1)
  unmounted "/compat/ubuntu/home" && (mount -t nullfs /home "/compat/ubuntu/home" || exit 1)
  unmounted "/compat/ubuntu/proc" && (mount -o nocover -t linprocfs linprocfs "/compat/ubuntu/proc" || exit 1)
  unmounted "/compat/ubuntu/sys" && (mount -o nocover -t linsysfs linsysfs "/compat/ubuntu/sys" || exit 1)
  unmounted "/compat/ubuntu/tmp" && (mount -t nullfs /tmp "/compat/ubuntu/tmp" || exit 1)
  unmounted /dev/fd && (mount -o nocover -t fdescfs fdescfs /dev/fd || exit 1)
  unmounted /proc && (mount -o nocover -t procfs procfs /proc || exit 1)
  true
}

load_rc_config $name
run_rc_command "$1"
```

Скрипт выше включит Ubuntu Base System на машине FreeBSD. Поэтому, когда компьютер перезагрузится/перезапустит систему FreeBSD, он автоматически прочитает скрипт Ubuntu Base System выше. Давайте попробуем перезапустить Ubuntu Base System сейчас.

```
root@ns1:~ # service ubuntu restart
```

## Создайте каталог совместимости ZFS Datasheet Linux
ZFS Linux datasheets используются для создания снимков, удаления файлов и безопасного удаления всего содержимого тома без нарушения файловой системы каталога FreeBSD. Следуйте сценарию ниже, чтобы создать ZFS datasheet для Linux.

```
root@ns1:~ # zfs create -o compression=on -o mountpoint=/compat zroot/compat
root@ns1:~ # zfs snapshot -r zroot/compat@2022-04-22
```

Если таблица данных ZFS Linux была создана, продолжите созданием каталога для базовой системы Ubuntu.

```
root@ns1:~ # mkdir -p /compat/ubuntu/{dev/fd,dev/shm,home,proc,sys,tmp}
```

Активируйте базовую систему Ubuntu.

```
root@ns1:~ # service ubuntu restart
compat.linux.emul_path: /compat/ubuntu -> /compat/ubuntu
```

## Установка базовой системы Ubuntu в каталог совместимости Linux
Файл установки базовой системы Ubuntu — «debootstrap». Этот файл будет использоваться для загрузки и установки базовой системы Ubuntu путем указания цели Ubuntu, например, focal для Focal Fossa, которая является версией Ubuntu 20.04 LTS.

```
root@ns1:~ # pkg install linux-sublime-text4
root@ns1:~ # pkg install debootstrap
```

Загрузите и установите Ubuntu Base System в каталог совместимости Linux.

```
root@ns1:~ # debootstrap --arch=amd64 --no-check-gpg focal /compat/ubuntu
I: Configuring netcat-openbsd...
I: Configuring isc-dhcp-client...
I: Configuring debconf-i18n...
I: Configuring vim-tiny...
I: Configuring ca-certificates...
I: Configuring libapt-pkg6.0:amd64...
I: Configuring gir1.2-glib-2.0:amd64...
I: Configuring whiptail...
I: Configuring keyboard-configuration...
I: Configuring libpython3.8-stdlib:amd64...
I: Configuring python3.8...
I: Configuring libxml2:amd64...
I: Configuring libpython3-stdlib:amd64...
I: Configuring apt...
I: Configuring apt-utils...
I: Configuring python3...
I: Configuring python3-six...
I: Configuring python3-gi...
I: Configuring shared-mime-info...
I: Configuring python3-netifaces...
I: Configuring lsb-release...
I: Configuring python3-cffi-backend...
I: Configuring python3-pkg-resources...
I: Configuring python3-dbus...
I: Configuring python3-yaml...
I: Configuring netplan.io...
I: Configuring ubuntu-advantage-tools...
I: Configuring python3-nacl...
I: Configuring networkd-dispatcher...
I: Configuring python3-pymacaroons...
I: Configuring console-setup-linux...
I: Configuring console-setup...
I: Configuring kbd...
I: Configuring ubuntu-minimal...
I: Configuring libc-bin...
I: Configuring systemd...
I: Configuring ca-certificates...
I: Base system installed successfully.
```

Исправление динамических ссылок (интерпретатор ELF) с помощью символических ссылок.

```
root@ns1:~ # cd /compat/ubuntu/lib64/
root@ns1:/compat/ubuntu/lib64 # rm ./ld-linux-x86-64.so.2
root@ns1:/compat/ubuntu/lib64 # ln -s ../lib/x86_64-linux-gnu/ld-2.31.so ld-linux-x86-64.so.2
```

Перезапустите Ubuntu.

```
root@ns1:/compat/ubuntu/lib64 # service linux restart
root@ns1:/compat/ubuntu/lib64 # service ubuntu restart
compat.linux.emul_path: /compat/ubuntu -> /compat/ubuntu
```

## Конфигурация Ubuntu
Чтобы настроить Ubuntu в системе FreBSD, мы должны войти в chroot jail, который ограничит процесс настройки базовой файловой системой Ubuntu. Если Ubuntu выводит сообщение об отсутствующем идентификаторе группы, это можно проигнорировать. Вы увидите, что командная строка изменится.

```
root@ns1:~ # chroot /compat/ubuntu /bin/bash
root@ns1:/# printf "%b\n" "0.0 0 0.0\n0\nUTC" > /etc/adjtime
root@ns1:/# dpkg-reconfigure tzdata

Current default time zone: 'Asia/Jakarta'
Local time is now:      Sun Jul  9 14:31:08 WIB 2023.
Universal Time is now:  Sun Jul  9 07:31:08 UTC 2023.

root@ns1:/# printf "APT::Cache-Start 251658240;" > /etc/apt/apt.conf.d/00aptitude
root@ns1:/# printf "APT::Cache-Start 251658240;" > /etc/apt/apt.conf.d/00aptitude
root@ns1:/# printf "deb http://archive.ubuntu.com/ubuntu/ focal main restricted universe multiverse" > /etc/apt/sources.list
root@ns1:/# exit
exit
root@ns1:~ #
```

Если скрипт выше был выполнен, это означает, что мы включили Linux Binary Compatibility на FreeBSD и установили на него базовую систему Ubuntu. С Ubuntu Linux, активным на FreeBSD, теперь мы можем установить репозиторий Ubuntu на FreeBSD. Теперь мы проверим, обновив Ubuntu с помощью базового скрипта Ubuntu, а именно "apt update".

```
root@ns1:~ # chroot /compat/ubuntu /bin/bash
root@ns1:/# apt update
Get:1 http://archive.ubuntu.com/ubuntu focal InRelease [265 kB]
Get:2 http://archive.ubuntu.com/ubuntu focal/main amd64 Packages [970 kB]
Get:3 http://archive.ubuntu.com/ubuntu focal/main Translation-en [506 kB]
Get:4 http://archive.ubuntu.com/ubuntu focal/restricted amd64 Packages [22.0 kB]
Get:5 http://archive.ubuntu.com/ubuntu focal/restricted Translation-en [6212 B]
Get:6 http://archive.ubuntu.com/ubuntu focal/universe amd64 Packages [8628 kB]
Get:7 http://archive.ubuntu.com/ubuntu focal/universe Translation-en [5124 kB]                                      
Get:8 http://archive.ubuntu.com/ubuntu focal/multiverse amd64 Packages [144 kB]                                     
Get:9 http://archive.ubuntu.com/ubuntu focal/multiverse Translation-en [104 kB]                                     
Fetched 15.8 MB in 9s (1715 kB/s)                                                                                   
Reading package lists... Done
Building dependency tree... Done
All packages are up to date.
```

Из скрипта выше видно, что все работает нормально. Продолжаем снова, обновляя Ubuntu.

```
root@ns1:/# apt list --upgradable
Listing... Done
root@ns1:/# apt upgrade
Reading package lists... Done
Building dependency tree... Done
Calculating upgrade... Done
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
root@ns1:/# apt autoremove
Reading package lists... Done
Building dependency tree... Done
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
root@ns1:/# apt clean
root@ns1:/#
```

Отлично, все конфигурации в порядке, и Ubuntu нормально работает на машине FreeBSD. Теперь вы можете устанавливать программы на основе Ubuntu на FreeBSD. Для большей убедительности мы тестируем установку приложения NGINX на Ubuntu.

```
root@ns1:~ # chroot /compat/ubuntu /bin/bash
root@ns1:/# apt update
Hit:1 http://archive.ubuntu.com/ubuntu focal InRelease
Reading package lists... Done
Building dependency tree       
Reading state information... Done
All packages are up to date.
root@ns1:/# apt install nginx
Reading package lists... Done
Building dependency tree... Done
The following additional packages will be installed:
  fontconfig-config fonts-dejavu-core libfontconfig1 libfreetype6 libgd3 libjbig0 libjpeg-turbo8 libjpeg8
  libnginx-mod-http-image-filter libnginx-mod-http-xslt-filter libnginx-mod-mail libnginx-mod-stream libpng16-16
  libtiff5 libwebp6 libx11-6 libx11-data libxau6 libxcb1 libxdmcp6 libxpm4 libxslt1.1 nginx-common nginx-core
Suggested packages:
  libgd-tools fcgiwrap nginx-doc ssl-cert
The following NEW packages will be installed:
  fontconfig-config fonts-dejavu-core libfontconfig1 libfreetype6 libgd3 libjbig0 libjpeg-turbo8 libjpeg8
  libnginx-mod-http-image-filter libnginx-mod-http-xslt-filter libnginx-mod-mail libnginx-mod-stream libpng16-16
  libtiff5 libwebp6 libx11-6 libx11-data libxau6 libxcb1 libxdmcp6 libxpm4 libxslt1.1 nginx nginx-common nginx-core
0 upgraded, 25 newly installed, 0 to remove and 0 not upgraded.
Need to get 3851 kB of archives.
After this operation, 12.8 MB of additional disk space will be used.
Do you want to continue? [Y/n] y
```

Чтобы установить приложения Ubuntu на машину FreeBSD, вам следует обратить внимание на изменение командной строки. Рассмотрим следующий пример.

```
root@ns1:~ #   
root@ns1:~ # chroot /compat/ubuntu /bin/bash
root@ns1:/#
```

Скрипт выше объясняет, что root@ns1:~ # — это командная строка FreeBSD, вы не можете запускать приложения Ubuntu в этой командной строке. После запуска команды chroot /compat/ubuntu /bin/bash командная строка изменится на root@ns1:/#, что означает, что мы активны в командной строке Ubuntu и можем устанавливать приложения Ubuntu в командной строке root@ns1:/#.
