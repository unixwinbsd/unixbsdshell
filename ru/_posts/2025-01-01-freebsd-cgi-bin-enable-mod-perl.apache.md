---
title: FreeBSD CGI BIN Включение mod PERL на Apache24
date: "2025-01-01 09:01:00 +0300"
id: freebsd-cgi-bin-enable-mod-perl.apache
lang: ru
layout: single
author_profile: true
categories:
  - FreeBSD
tags: "WebServer"
excerpt: "Моды perl, установленные на FreeBSD, — это больше, чем просто скрипты CGI на стероидах. Моды perl — это новый способ создания динамического контента"
keywords: freebsd, apache, cgi, bin, web, mod, perl
---

Perl mod — это модуль языка программирования Perl, который вставляется в веб-сервер Apache24. Моды Perl можно использовать для управления веб-сервером Apache24, ответа на запросы веб-страниц и многого другого.

Моды Perl, установленные на FreeBSD, — это больше, чем просто скрипты CGI на стероидах. Моды Perl — это новый способ создания динамического контента, использующий всю мощь веб-сервера Apache для сохранения состояния веб-сайта. Настраиваемая система аутентификации пользователей, более мощное использование прокси-сервера и многое другое. Однако, как ни странно, ваш старый скрипт CGI продолжит работать и работать очень быстро. С модом Perl вы получите больше преимуществ от производительности мода Perl, интегрированного в Apache24.

В этой статье объясняется, как включить моды Perl на Apache24. В этой статье моды Perl и Apache24 будут установлены одновременно на системе FreeBSD 13.2.

## 1. Установка CGI-мод Apache24 Perl
Чтобы использовать функцию мода Perl, сначала необходимо установить веб-сервер Apache24. Ниже приведено руководство по установке модов Perl на Apache24.

```
root@ns1:~ # cd /usr/ports/www/apache24
root@ns1:/usr/ports/www/apache24 # make install clean
```

После успешной установки apache24 можно продолжить установку модуля perl.

```
root@ns1:~ # cd /usr/ports/www/mod_perl2
root@ns1:/usr/ports/www/mod_perl2 # make install clean
root@ns1:~ # cd /usr/ports/databases/p5-DBI
root@ns1:/usr/ports/databases/p5-DBI # make install clean
root@ns1:~ # cd /usr/ports/www/p5-Apache-DBI
root@ns1:/usr/ports/www/p5-Apache-DBI # make install clean
```

Откройте файл /usr/local/etc/apache24/httpd.conf, активируйте ServerName, удалив знак «#» перед скриптом.

#ServerName www.example.com:80

Изменить с

ServerName www.unixexplore.com:80

www.example.com меняется на доменное имя на вашем сервере FreeBSD, в этом случае доменное имя, которое я установил в файле /etc/hosts, это unixexplore.com. Если вы не указали доменное имя в файле /etc/hosts, пожалуйста, создайте доменное имя, обратите внимание на следующий пример записи доменного имени в файле /etc/hosts.

```
root@ns1:~ # ee /etc/hosts
::1                             localhost   localhost.unixexplore.com
127.0.0.1                       localhost   localhost.unixexplore.com
192.168.5.2                     ns1            ns1.unixexplore.com
192.168.5.2                     www.unixexplore.com
```

Все еще в файле /usr/local/etc/apache24/httpd.conf, после скрипта.

```
<Directory "/usr/local/www/apache24/cgi-bin">
    AllowOverride None
    Options None
    Require all granted
</Directory>
```

Ниже вы добавляете следующий скрипт.

```
<Location /cgi-bin/*.pl>
    SetHandler perl-script
    PerlResponseHandler ModPerl::PerlRun
    PerlSendHeader On
    Options ExecCGI
    Require all granted
</Location>
<Location /cgi-bin/*.cgi>
    SetHandler perl-script
    PerlResponseHandler ModPerl::PerlRun
    PerlSendHeader On
    Options ExecCGI
    Require all granted
</Location>
```

Теперь откройте файл /usr/local/etc/apache24/modules.d/260_mod_perl.conf и активируйте модуль perl, удалив знак «#» в следующем скрипте.

#LoadModule perl_module        libexec/apache24/mod_perl.so

Remove the sign "#" thus becoming

LoadModule perl_module        libexec/apache24/mod_perl.so


После завершения настройки Apache24 введите следующий скрипт в файл /etc/rc.conf.

```
root@ns1:~ # ee /etc/rc.conf
apache24_enable="YES"
```

Перезапустите веб-сервер Apache24.

```
root@ns1:~ # service apache24 restart
```

## 2. Тест Apache24 Perl mod CGI
Чтобы протестировать этот Perl mod, мы создадим тестовый файл с именем "test.cgi", который будет помещен в папку /usr/local/www/apache24/cgi-bin. Вот как создать файл.

```
root@ns1:~ # touch /usr/local/www/apache24/cgi-bin/test.cgi
```

Дайте разрешения файлу /usr/local/www/apache24/cgi-bin/test.cgi.

```
root@ns1:~ # chmod 755 /usr/local/www/apache24/cgi-bin/test.cgi
```

Предоставьте права собственности на файл /usr/local/www/apache24/cgi-bin/test.cgi или папку /usr/local/www/apache24/.

```
root@ns1:~ # chown -R www:www /usr/local/www/apache24/
root@ns1:~ # chown -R www:www /usr/local/www/apache24/cgi-bin/test.cgi
```

Теперь откройте файл /usr/local/www/apache24/cgi-bin/test.cgi и введите приведенный ниже скрипт. Используйте редактор «ee» или редактор «nano» для ввода следующего скрипта.

```
root@ns1:~ # ee /usr/local/www/apache24/cgi-bin/test.cgi
#!/usr/local/bin/perl
print "Content-Type: text/html; charset=utf-8 \n\n";
print "<h1>Congratulations on successfully configuring the Perl mod on Apache24!</h1>";
```

Перезапустите веб-сервер Apache24.

```
root@ns1:~ # service apache24 restart
```

После перезапуска проведите тест, открыв браузер Яндекс или Google Chrome. Введите в браузере следующую команду.

```
http://192.168.5.2/cgi-bin/test.cgi
```

Помните, IP 192.168.5.2 — это IP частного сервера FreeBSD. Если нет неправильных конфигураций, появится сообщение:

Congratulations on successfully configuring the Perl mod on Apache24!



