---
title: "Установка DNS-сервера ISC Bind с использованием OpenBSD"
date: 2024-12-01 17:00:00+03:00
excerpt: "Принцип работы кэширующего DNS-сервера заключается в том, что когда DNS-сервер находит запрошенный вами DNS-адрес, он немедленно отправляет его тому, кто запросил DNS-адрес, а затем сохраняет DNS-адрес в своей базе данных."
id: install-isc-dns-bind-on-openbsd
lang: ru
layout: single
author_profile: true
categories:
  - OpenBSD
tags: "DNSServer"
keywords: isc, dns, bind, openbsd, configuration, server, openssl, http, https, ssl
---

Для тех из вас, у кого медленный интернет, кэширование DNS-сервера является разумным решением для увеличения скорости интернет-серфинга. В общем случае приложение DNS-сервера выполняет функцию преобразования имен в числа. Каждое введенное вами имя веб-сайта будет преобразовано в IP-адрес, поэтому веб-страница будет отображаться так, как вы ее видите в Google Chrome или Firefox.

В целом существует несколько типов DNS-серверов. Кэширующий тип — это тип, который не содержит необработанного сопоставления имен с адресами. Этот тип называется авторитетным DNS-сервером, который можно разделить на главный, подчиненный и скрытый. Кэширующий DNS-сервер также называется рекурсивным сервером. Все типы DNS-серверов могут называться серверами имен.

Принцип работы кэширующего сервера DNS заключается в том, что когда сервер DNS находит запрошенный вами адрес DNS, он немедленно отправляет его тому, кто запросил адрес DNS, а затем сохраняет адрес DNS в своей базе данных. Позже, когда ваш друг запросит тот же DNS-адрес, сервер кэширования DNS возьмет DNS-адрес из своей базы данных. Таким образом, это может ускорить время обработки DNS-запросов.

Чем медленнее ваше интернет-соединение, тем полезнее кэш DNS-сервера для улучшения доступа в интернет. Для поддержания точности на сервере существует настраиваемое время истечения срока действия (время жизни/TTL), которое заставляет его периодически обращаться в Интернет за обновлениями.

## 1. Как установить DNS-сервер ISC Bind

Одним из преимуществ ISC Bind является то, что его можно установить на все операционные системы, такие как Windows, MacOS, Linux и BSD, включая OpenBSD. Однако каждая система имеет свой способ настройки. В этом разделе мы узнаем, как установить, настроить и использовать ISC Bind на сервере OpenBSD.

Поскольку DNS Bind уже хорошо знаком, как и в других операционных системах, в OpenBSD ISC Bind уже доступен в репозитории pkg_add. Для установки ISC Bind на OpenBSD вы можете выполнить команду ниже.

```
foo# pkg_add isc-bind-9.18.25v3
```

После завершения процесса установки приложение ISC Bind по умолчанию создает пользователя и группу с именем _bind:_bind. Этот пользователь будет очень полезен при настройке. Расположение каталога ISC Bind в OpenBSD немного отличается от расположения в FreeBSD. OpenBSD помещает приложение ISC Bind в /var/named. Все файлы конфигурации хранятся в этом каталоге.

По умолчанию каталог /var/named содержит две папки, а именно /etc и /tmp, в процессе настройки мы добавим в этот каталог несколько папок, таких как /master, /standard и другие.

## 2. Процесс настройки

В этом разделе мы обсудим шаги по настройке ISC Bind. Файл конфигурации ISC Bind называется «named.conf». В этом файле вы можете задать все необходимые параметры кэширования DNS-сервера.

### а. Включение привязки ISC
Перед настройкой ISC Bind сначала включите DNS-сервер Bind, выполнив команду ниже.

```
foo# rcctl enable isc_named
foo# rcctl restart isc_named
isc_named(ok)
isc_named(ok)
```

Приведенная выше команда включает ISC Bind и создает файл rndc.key. Попробуйте открыть содержимое файла скрипта rndc.key с помощью команды cat.

```
foo# cat /var/named/etc/rndc.key
key "rndc-key" {
        algorithm hmac-sha256;
        secret "sLhSyJAo609lksFnU2Z0y5MbiSnoVJfTMz21foPVv3g=";
};
```

### б. Отредактируйте файл конфигурации с именем .conf.
Мы будем использовать все содержимое файла скрипта rndc.key для настройки файла named.conf. Теперь откройте файл named.conf, затем удалите все содержимое скрипта и замените его скриптом, приведенным ниже.

```
foo# nano /var/named/etc/named.conf
key "rndc-key" {
	algorithm hmac-sha256;
	secret "sLhSyJAo609lksFnU2Z0y5MbiSnoVJfTMz21foPVv3g=";
};

 
 controls {
 	inet 127.0.0.1 port 953
 		allow { 127.0.0.1; } keys { "rndc-key"; };
 };

acl clients {
	192.168.7.0/24;
	127.0.0.1;
};
acl IP_LAN { 192.168.7.3; };

options {
	directory "/tmp";
	version "dns foo.kursor.my.id";
	listen-on port 53 { IP_LAN; };
	#listen-on-v6 { any; };
	allow-recursion { clients; };
	allow-query { clients; };
	allow-query-cache { clients; };
        allow-transfer { none; };
	empty-zones-enable yes;
	recursion yes;
	auth-nxdomain no;
	dnssec-validation yes;
};

zone "localhost" {
        type master;
        file "standard/localhost";
        allow-transfer { 127.0.0.1; };
};

zone "127.in-addr.arpa" {
        type master;
        file "standard/loopback";
        allow-transfer { 127.0.0.1; };
};

zone "." {
  type forward;
  forward first;
  forwarders { 1.1.1.1; 8.8.8.8; };
};

zone "kursor.my.id" in {
        type master;
        file "master/internal.lan";
};

zone "7.168.192.in-addr.arpa" in {
        type master;
        file "master/insternal.rev";
};
```

### в. Добавление настроек зоны
В приведенном выше скрипте файла named.conf есть несколько зон, которые будут использоваться ISC Bind. Мы настроим создание зоны согласно приведенному выше сценарию. Первым шагом является создание каталога для зоны.

```
foo# mkdir -p /var/named/standard
foo# mkdir -p /var/named/master
foo# touch /var/named/standard/localhost
foo# touch /var/named/standard/loopback
foo# touch /var/named/master/internal.lan
foo# touch /var/named/master/internal.rev
```

После этого в файл зоны добавляется скрипт, как в примере ниже. Как обычно, мы используем текстовый редактор nano.

```
foo# nano /var/named/standard/localhost
$ORIGIN localhost.
$TTL 6h

@       IN      SOA     localhost. root.localhost. (
                        1       ; serial
                        1h      ; refresh
                        30m     ; retry
                        7d      ; expiration
                        1h )    ; minimum

                NS      localhost.
                A       127.0.0.1
                AAAA    ::1
```

```
foo# nano /var/named/standard/loopback
$ORIGIN 127.in-addr.arpa.
$TTL 6h

@       IN      SOA     localhost. root.localhost. (
                        1       ; serial
                        1h      ; refresh
                        30m     ; retry
                        7d      ; expiration
                        1h )    ; minimum

                NS      localhost.
1.0.0           PTR     localhost.
```

```
foo# nano /var/named/master/internal.lan
$ORIGIN .
$TTL    86400   ; 24 hours
kursor.my.id    IN SOA  foo.kursor.my.id. root.foo.kursor.my.id. (
                        2010022201 ; Serial
                        86400           ; Refresh (24 hours)
                        3600             ; Retry (1 hour)
                        172800         ; Expire (48 hours)
                        3600             ; Minimum (1 hour)
                )
kursor.my.id.		IN	NS      foo.kursor.my.id.

$ORIGIN kursor.my.id.
foo.kursor.my.id.	IN	A       192.168.7.3
```

```
foo# nano /var/named/master/insternal.rev
$ORIGIN .
$TTL    86400   ; 24 hours
7.168.192.in-addr.arpa	IN	SOA	foo.kursor.my.id. root.foo.kursor.my.id. (
                                2010022201      ; Serial
                                86400           ; Refresh (24 hours)
                                3600            ; Retry (1 hour)
                                172800          ; Expire (48 hours)
                                3600            ; Minimum (1 hour)
                                )
	IN	NS      foo.kursor.my.id.

$ORIGIN 7.168.192.in-addr.arpa.
100	IN     PTR     foo.kursor.my.id.
```

### г. Изменение разрешений и права собственности
Как мы объяснили выше, по умолчанию ISB Bind, работающий в OpenBSD, имеет пользователя и группу «_bind». Выполните команду ниже, чтобы узнать права доступа к файлу и права собственности.

```
foo# cd /var/named
foo# chown -R _bind:_bind master standard var tmp
```

### е. Файл корневых ссылок
Файл корневых подсказок — это файл, содержащий имена и IP-адреса авторитетных серверов имен для корневой зоны, чтобы программное обеспечение могло инициировать процесс разрешения DNS. Вам следует загрузить этот файл из официального репозитория сайта Iana.

```
foo# wget https://www.internic.net/domain/named.root -P /var/named/etc
```

### ф. Отредактируйте файл resolv.conf
Файл resolv.conf в OpenBSD используется для подключения сервера OpenBSD к системе доменных имен (DNS). Вам необходимо будет заполнить этот файл DNS, который вы будете использовать. Поскольку вы используете ISC Bind в качестве кэширующего DNS-сервера, мы заполним этот файл частными IP-адресами серверов OpenBSD.

```
foo# nano /etc/resolv.conf
domain kursor.my.id
nameserver 192.168.7.3
```

### г. Зона проверки
Чтобы убедиться в отсутствии ошибок в созданных вами зонах, выполните проверку с помощью команды ниже. Сначала мы проверяем основной файл конфигурации с именем .conf.

```
foo# named-checkconf /var/named/etc/named.conf
```

Затем вы переходите к проверке зон, которые вы создали выше.

```
foo# named-checkzone kursor.my.id /var/named/master/internal.lan
zone kursor.my.id/IN: loaded serial 2010022201
OK
foo# named-checkzone 7.168.192.in-addr.arpa /var/named/master/internal.rev
zone 7.168.192.in-addr.arpa/IN: loaded serial 2010022201
OK
```

### час Проверьте DNS-серверы имен
Если все в порядке, продолжаем проверку сервера имен DNS. Данная проверка призвана гарантировать правильную работу DNS-сервера ISB Bind. Мы используем команду dig для проверки сервера имен DNS. Взгляните на примеры команд ниже.

```
foo# dig yahoo.com

; <<>> dig 9.10.8-P1 <<>> yahoo.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 59861
;; flags: qr rd ra; QUERY: 1, ANSWER: 6, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;yahoo.com.                     IN      A

;; ANSWER SECTION:
yahoo.com.              1800    IN      A       74.6.231.21
yahoo.com.              1800    IN      A       98.137.11.163
yahoo.com.              1800    IN      A       74.6.143.25
yahoo.com.              1800    IN      A       74.6.231.20
yahoo.com.              1800    IN      A       98.137.11.164
yahoo.com.              1800    IN      A       74.6.143.26

;; Query time: 27 msec
;; SERVER: 192.168.7.3#53(192.168.7.3)
;; WHEN: Wed Apr 17 16:40:48 WIB 2024
;; MSG SIZE  rcvd: 134
```

Вот несколько команд dig, которые можно использовать для проверки DNS-серверов ISC Bind.

```
foo# dig @192.168.7.3 azion.com
foo# dig facebook.com +trace
foo# dig -x 172.217.14.238
foo# dig google.com +short
forcesafesearch.google.com.
216.239.38.120
```

Помимо команды dig, вы также можете использовать команду nslookup. Ниже мы приводим пример использования nslookup.

```
foo# nslookup -type=ns google.com
Server:         192.168.7.3
Address:        192.168.7.3#53

Non-authoritative answer:
google.com      nameserver = ns2.google.com.
google.com      nameserver = ns3.google.com.
google.com      nameserver = ns4.google.com.
google.com      nameserver = ns1.google.com.

Authoritative answers can be found from:
ns1.google.com  internet address = 216.239.32.10
ns2.google.com  internet address = 216.239.34.10
ns3.google.com  internet address = 216.239.36.10
ns4.google.com  internet address = 216.239.38.10
ns1.google.com  has AAAA address 2001:4860:4802:32::a
ns2.google.com  has AAAA address 2001:4860:4802:34::a
ns3.google.com  has AAAA address 2001:4860:4802:36::a
ns4.google.com  has AAAA address 2001:4860:4802:38::a
```

```
foo# nslookup -type=mx yahoo.com
foo# nslookup -type=soa facebook.com
foo# nslookup -type=txt google.com
foo# nslookup google.com ns1.google.com
```

Следуя этому руководству, вы сможете использовать ISB Bind в качестве своего личного DNS-сервера. Вы можете использовать его на своем компьютере под управлением Windows, установив IP-адрес DNS-сервера на IP-адрес 192.168.7.3. Теперь оцените скорость доступа в Интернет с помощью DNS-сервера ISC Bind. Вы наверняка почувствуете разницу.


