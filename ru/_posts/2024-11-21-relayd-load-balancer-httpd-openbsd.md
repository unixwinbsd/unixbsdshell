---
title: "Балансировщик нагрузки Relayd с httpd на OpenBSD"
date: 2024-11-21 19:00:00+03:00
excerpt: "Relayd — это демон для динамической пересылки и маршрутизации входящих соединений к хостам назначения. Его основная цель — выступать в качестве балансировщика нагрузки, шлюза прикладного уровня или прозрачного прокси-сервера."
id: relayd-load-balancer-httpd-openbsd
lang: ru
---

Балансировщик нагрузки или система — это устройство, которое может перенаправлять входящий трафик на один или несколько реальных серверов. По своему назначению он похож на простое пересылающее реле. Коммерческие решения, такие как Cisco Local Director, предоставляют расширенные функции и набор алгоритмов, которые позволяют выбрать подходящий сервер, включая опрос доступности кандидатов или доступности агентов/клиентов. Из-за особенностей реализации их часто называют машинами обратного NAT или обратными прокси-серверами.

Relayd — это демон для динамической пересылки и маршрутизации входящих соединений к хостам назначения. Его основная цель — выступать в качестве балансировщика нагрузки, шлюза прикладного уровня или прозрачного прокси-сервера. Демон может контролировать доступность группы хостов, которая определяется путем проверки определенных служб, которые являются общими для этой группы хостов. После подтверждения доступности настраиваются службы пересылки уровня 3 и/или уровня 7 посредством повторной передачи.

Этот репозиторий включает в себя копию исходного дерева потокового кода, которая время от времени обновляется, а также экспериментальные ветви, которыми следует поделиться с другими. Это не переносимая версия и предназначена только для OpenBSD! Этот репозиторий может содержать в значительной степени экспериментальные изменения; не используйте его в производстве. Основные разработки происходят в дереве CVS OpenBSD.

Для обеспечения доступности веб-сайтов в OpenBSD предусмотрено несколько функциональных решений, которые отлично работают вместе; relayd и httpd.

## 1. Что такое ретрансляция

Relayd — это балансировщик нагрузки с открытым исходным кодом, способный обрабатывать протоколы уровней 3, 4 и 7. Relayd, ранее известный как hoststated, — это новейшее приложение для балансировки нагрузки, разработанное группой OpenBSD. Relayd можно настроить как прямой, обратный или ретранслятор типа TCP-порта или перенаправления, а также использовать в качестве ускорителя SSL. Relayd — это быстрый, безопасный и стабильный интерфейс для веб-сервера или веб-кластера.

Httpd — веб-сервер, официально поддерживаемый и разрабатываемый OpenBSD. Около 20 лет назад OpenBSD впервые импортировала модифицированный сервер Apache httpd в свою базовую версию. Позже, по мере роста популярности nginx, после его выпуска он в конечном итоге стал веб-сервером, хотя nginx также был исправлен для соответствия более строгим правилам безопасности OpenBSD. Затем, в 2015 году, OpenBSD выпустила собственный httpd, основанный на Relayd, в качестве веб-сервера по умолчанию.

Relayd и httpd могут работать вместе, обеспечивая простое, но высокодоступное решение для веб-хостинга и многого другого. В этой статье будет показано, как развернуть две виртуальные машины OpenBSD httpd, работающие за виртуальной машиной OpenBSD, используя реле в качестве балансировщика нагрузки. Два сервера httpd должны быть доступны только через балансировку нагрузки, а конфигурация должна быть способна обрабатывать ситуации, когда один из двух серверов httpd может оказаться недоступным во время исправления или перезапуска. Конфигурация также должна иметь рейтинг A на ssllabs.com.

Ниже приведены основные функции relayd:
1. Демон общего назначения доступен в OpenBSD начиная с версии 4.3*:
- балансировщик нагрузки.
- шлюз прикладного уровня.
- прозрачный прокси.
2. Возможность мониторинга групп хостов на предмет высокой доступности.
3. Функционирует как:
- Перенаправление уровня 3 через связь с pf.
- Ретрансляция уровня 7 с фильтрацией на уровне приложений.

## 2. Настройте relayd.conf

Первый шаг, который вам необходимо сделать, — это настроить файл relayd.conf. Ниже приведен пример скрипта relayd.conf.

Чтобы в любой момент отобразить набор заблокированных адресов в таблице SSHguard, введите следующую команду.

```
ext_addr = “1.1.1.1”
web_srvr = “10.1.1.10”
log state changes
log connection
http protocol httpFilter {
 #return error
 http headerlen 4096
 match request header set “X-Forwarded-For” value “$REMOTE_ADDR”
 match request header set “X-Forwarded-SPort” value “$REMOTE_PORT”
 match request header set “X-Forwarded-DPort” value “$SERVER_PORT”
 match response header remove “Server” value “*”
 match header log “Host”
 match header log “X-Forwarded-For”
 match header log “User-Agent”
 match header log “X-Req-Status”
 match url log
 match request tag “BAD_METHOD”
 match request method GET tag “OK_METH”
 match request method HEAD tag “OK_METH”
 block request quick tagged “BAD_METHOD”
 match request header “Host” value “domain.com” tag “OK_REQ”
 match request header “Host” value “www.domain.com" tag “OK_REQ”
 match request header “Host” value “another.domain.com” tag “OK_REQ”
 block request quick tagged “OK_METH” tag “BAD_HH”
 block request quick path “*.php” tag NO_PHP
 block request quick path “*.cgi” tag NO_CGI
 block request quick path “*.js” tag NO_JS
 block request quick path “*.asp” tag NO_ASP
 block tag “BAD_REQ”
 pass request tagged “OK_REQ”
}
http protocol httpsFilter {
 #return error
 http headerlen 4096
 tls ciphers “ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA:ECDHE-ECDSA-AES128-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA256”
 tls edh
 match request header set “X-Forwarded-For” value “$REMOTE_ADDR”
 match request header set “X-Forwarded-SPort” value “$REMOTE_PORT”
 match request header set “X-Forwarded-DPort” value “$SERVER_PORT”
 match response header remove “Server” value “*”
 match header log “Host”
 match header log “X-Forwarded-For”
 match header log “User-Agent”
 match header log “X-Req-Status”
 match url log
 match response header set “Strict-Transport-Security” value “max-age=63072000; includeSubdomains; preload”
 match request tag “BAD_METHOD”
 match request method GET tag “OK_METH”
 match request method HEAD tag “OK_METH”
 block request quick tagged “BAD_METHOD”
 match request header “Host” value “domain.com” tag “OK_REQ”
 match request header “Host” value “www.domain.com" tag “OK_REQ”
 match request header “Host” value “another.domain.com” tag “OK_REQ”
 block request quick tagged “OK_METH” tag “BAD_HH”
 block request quick path “*.php” tag NO_PHP
 block request quick path “*.cgi” tag NO_CGI
 block request quick path “*.js” tag NO_JS
 block request quick path “*.asp” tag NO_ASP
 block tag “BAD_REQ”
 pass request tagged “OK_REQ”
}
relay www {
 listen on $ext_addr port 80
 protocol httpFilter
 forward to $web_srvr port 80
}
relay wwwssl {
 listen on $ext_addr port 443 tls
 protocol httpsFilter
 forward to $web_srvr port 80
}
```

В качестве IP-адреса мы рекомендуем использовать localhost, чтобы relayd прослушивал порт 127.0.0.1, и использовать правило rdr-to в pf. Внутренний веб-сервер использует адреса RFC1918.

```
ext_addr = "127.0.0.1"
web_srvr = "10.1.1.10"
```

Полезно для записи дополнительной информации и идентификации тех, кто тестирует конфигурации TLS.

```
log state changes
log connection
```

Вот пример конфигурации файла httpd.

```
server "example.com" {
    alias "chat.example.com"
    listen on * port 80
    location "/.well-known/acme-challenge/*" {
            root "/acme"
            request strip 2
    }
    location * {
            block return 301 "https://$HTTP_HOST$REQUEST_URI"
    }
}

server "example.com" {
    listen on * port 8080
    location * {
            root "/htdocs/www/public/"
    }
}
```

В этом заключительном разделе мы кратко рассмотрим настройку балансировщика нагрузки для приема http- и https-соединений и пересылки запросов на внутренний http-сервер. Вы должны иметь возможность безопасно исправить и перезагрузить один из серверов httpd, не теряя при этом доступа клиентов ни к одному из сайтов. Однако еще одно замечание: было бы разумно заблокировать pf на балансировщике нагрузки, если вы сочтете это целесообразным, поскольку он также действует как брандмауэр для ваших двух серверов httpd.
