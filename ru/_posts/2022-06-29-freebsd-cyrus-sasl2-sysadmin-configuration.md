---
title: FREEBSD SYSADMIN Установка и настройка Cyrus-Sasl2
date: "2022-06-29 18:15:00 +0300"
excerpt: "Добавление поддержки аутентификации в Postfix важно для пользователей, которые хотят ретранслировать электронную почту через свои серверы из незащищенных публичных сетей."
id: freebsd-cyrus-sasl2-sysadmin-configuration
lang: ru
layout: single
author_profile: true
categories:
  - FreeBSD
keywords: cyrus, sasl2, sasl, mail, server, email, freebsd
tags: "SysAdmin"
---

SASL (Simple Authentication and Security Layer) — это система для добавления поддержки аутентификации на основе соединения с протоколом. В этой статье мы будем использовать механизм Simple Mail Transfer Protocol (SMTP), а именно Postfix MTA использует SMTP для передачи интернет-почты. Добавление поддержки аутентификации в Postfix важно для пользователей, которые хотят ретранслировать почту через свои серверы из незащищенных публичных сетей. Безопасная ретрансляция почты может быть достигнута путем объединения SASL с шифрованием на основе SSL/TLS.

По умолчанию Postfix MTA не является открытым ретранслятором электронной почты. Хотя это не позволяет неавторизованным пользователям использовать сервер для доставки спама, это также не позволяет авторизованным пользователям отправлять электронную почту из мест, отличных от локальной частной сети. Cyrus SASL позволяет серверам SMTP проверять личность удаленных пользователей. После аутентификации пользователю предоставляется привилегия удаленной ретрансляции.

Джон Майерс, бывший системный архитектор в Университете Карнеги-Меллона, опубликовал спецификацию SASL в октябре 1997 года. Cyrus SASL поддерживается в рамках проекта Cyrus в Карнеги-Меллоне.

Обсуждение в этой статье особенно актуально для конфигураций Cyrus SASL с Postfix MTA и предполагает, что вы собираетесь использовать SASL с SMTP в сочетании с шифрованием SSL/TLS для защиты методов аутентификации PLAIN и/или LOGIN.

Операционная система, используемая в этой статье, использует FreeBSD 13. Вы можете начать установку сервера аутентификации Cyrus SASL, который включает Cyrus SASL. Чтобы начать установку сервера аутентификации Cyrus SASL, введите следующую команду:

```
root@router2:~ # cd /usr/ports/security/cyrus-sasl2-saslauthd
root@router2:~ # make config && make install clean
root@router2:~ # rehash
```

Появится меню с параметрами для запуска процесса установки Cyrus SASL. Просто оставьте эти настройки по умолчанию и нажмите кнопку «Ввод».

![Menu cyrus-sasl-sslauthd](/img/Menu cyrus-sasl-sslauthd.jpg)

После завершения установки security/cyrus-sasl2 отредактируйте файл /usr/local/lib/sasl2/Sendmail.conf или создайте его, если он не существует, и добавьте следующие строки:

```
pwcheck_method: saslauthd
mech_list: plain login
```

Скрипт выше поясняет, что первая строка указывает Cyrus SASL использовать установленный вами сервер аутентификации SASL. Вторая строка указывает Cyrus SASL объявлять только методы PLAIN и LOGIN, когда клиент изначально подключается к SMTP-серверу.

Следующий шаг — включить автоматический запуск сервера аутентификации SASL при загрузке компьютера. Чтобы настроить автоматический запуск сервера аутентификации SASL при загрузке, добавьте следующий скрипт в файл /etc/rc.conf.

```
saslauthd_enable="YES"
saslauthd_flags="-a pam"
```

Запустите демон saslauthd с помощью следующей команды.

```
root@router2:~ # service saslauthd restart
```

Этот демон функционирует как sendmail-брокер для аутентификации с помощью базы данных passwd на FreeBSD. Это избавит вас от необходимости создавать новый набор имен пользователей и паролей для каждого пользователя, которому необходимо использовать SMTP-аутентификацию, и сохранит логин и пароли электронной почты прежними.

Если демон saslauthd уже запущен, введите следующий скрипт в файл /etc/make.conf.

```
root@router2:~ # ee /etc/make.conf
SENDMAIL_CFLAGS=-I/usr/local/include/sasl -DSASL
SENDMAIL_LDFLAGS=-L/usr/local/lib
SENDMAIL_LDADD=-lsasl2
```

Скрипт выше дает Sendmail правильные параметры конфигурации для связи с cyrus-sasl2 во время компиляции. Убедитесь, что cyrus-sasl2 установлен перед повторной компиляцией Sendmail. Теперь мы продолжим повторную компиляцию Sendmail, выполнив следующую команду:

```
root@router2:~ # cd /usr/src/lib/libsmutil
root@router2:~ # make cleandir && make obj && make
root@router2:~ # cd /usr/src/lib/libsm
root@router2:~ # make cleandir && make obj && make
root@router2:~ # cd /usr/src/usr.sbin/sendmail
root@router2:~ # make cleandir && make obj && make && make install
```

После компиляции и переустановки Sendmail отредактируйте файл /etc/mail/freebsd.mc или локальный файл .mc. Многие администраторы предпочитают использовать вывод имени хоста в качестве имени файла .mc для уникальности. Добавьте эту строку в файл /etc/mail/freebsd.mc и поместите ее в конец файла.

```
root@router2:~ # ee /etc/mail/freebsd.mc
dnl set SASL options
TRUST_AUTH_MECH(`GSSAPI DIGEST-MD5 CRAM-MD5 LOGIN')dnl
define(`confAUTH_MECHANISMS', `GSSAPI DIGEST-MD5 CRAM-MD5 LOGIN')dnl
```

Последний шаг — запустить команду make в /etc/mail, скрипт ниже запустит новый файл расширения .mc и создаст файл расширения .cf с именем freebsd.cf. Введите скрипт ниже, чтобы скопировать файл в sendmail.cf.

```
root@router2:~ # cd /etc/mail
root@router2:/etc/mail # make install restart
```

Перекомпиляция Sendmail не должна вызвать проблем, если /usr/src не сильно изменился и доступны необходимые зависимости.


