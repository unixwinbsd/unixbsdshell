---
title: "FreeBSD Knot Resolver — реализация DNS Caching Resolver"
date: 2024-12-18 19:00:00+03:00
excerpt: "Knot DNS Resolver включает в себя полное приложение для реализации кэш-резолвера, написанное на LuaJIT и C."
id: freebsd-knot-resover-dns-caching-resolver
lang: ru
layout: single
author_profile: true
categories:
  - FreeBSD
tags: "DNSServer"
keywords: dns, knot, resolver, openbsd, configuration, httpd, openssl, http, https, isc
---

Knot Resolver — это кэширующий DNS-резолвер, который можно использовать в крупных сетях, например, у интернет-провайдеров, а также настоятельно рекомендуется для использования на маршрутизаторах домашних сетей. Knot Resolver — это современная реализация решателя, разработанная для масштабируемости, надежности и гибкости. Конструкция разрешителя узлов отличается от других разрешителей. Его базовая архитектура небольшая и эффективная, а большинство его функций можно реализовать в виде дополнительных модулей, что ограничивает поверхность атаки и повышает производительность Knot Resolver.

Knot DNS Resolver включает в себя полное приложение для реализации кэш-резолвера, написанное на LuaJIT и C. В Knot resolver есть много модулей, которые вы можете использовать, например, API-модули для расширений и многое другое. В целом существует три встроенных модуля: итератор, кэш, валидатор и множество других внешних модулей.

В модуле Lua кэш-преобразователь Knot является маршрутизируемым и общим, а быстрая привязка FFI делает его отличным для использования в процессе разрешения или для использования в вашей рекурсивной службе DNS. Это OpenResty DNS.

Сервер-резолвер DNS Knot использует иную стратегию масштабирования, чем другие рекурсивные DNS-серверы: он работает без потоковой передачи, не имеет общей архитектуры (за исключением общего кэша MVCC). Вы можете запускать и останавливать дополнительные узлы в зависимости от их загруженности без простоя.

В этой статье мы узнаем, как установить, настроить и использовать резолвер Knot на машине FreeBSD.

## A. Процесс установки решателя узлов

Как и большинство приложений, работающих на FreeBSD, он использует PKG и порты для процесса установки. Аналогично с устройством разрешения узлов вы можете использовать либо систему PKG, либо систему портов. Хотя процесс установки с помощью системы портов занимает много времени, установленные библиотеки весьма полны. Поэтому мы рекомендуем вам использовать портовую систему для начала установки решателя узлов.

Введите следующую команду, чтобы начать установку решателя узлов.

```
root@ns3:~ # cd /usr/ports/dns/knot-resolver
root@ns3:/usr/ports/dns/knot-resolver # make config
root@ns3:/usr/ports/dns/knot-resolver # make install clean
```

В команде «make config» появится меню опций, которые необходимо активировать. Если он активен, просто нажмите «ОК».

После запуска команды «make install clean» система портов FreeBSD автоматически выполнит процесс установки. Дождитесь завершения процесса.

Оказывается, установить Knot resolver довольно просто, это может сделать каждый. Оказывается, процесс на этом не останавливается. Остается процесс настройки и порядок его использования.

## B. Процесс настройки преобразователя узлов

Конфигурация — самый важный этап, вам предстоит изменять, добавлять и удалять скрипты, содержащиеся в файле конфигурации. По умолчанию каталогом резолвера Knot является «/usr/local/etc/knot-resolver», а его файл конфигурации называется «kresd.conf».

Откройте файл «kresd.conf», отредактируйте скрипт и настройте его в соответствии со спецификациями вашего сервера FreeBSD. В качестве руководства вы можете использовать скрипт «kresd.conf», как показано ниже.

```
net.listen('192.168.5.2', 53, { kind = 'dns' })


-- Load useful modules
modules = {
	'hints > iterate',  -- Allow loading /etc/hosts or custom root hints
	'stats',            -- Track internal statistics
	'predict',          -- Prefetch expiring/frequent records
}

internal_domains = policy.todnames({
  'datainchi.com.'
})

-- The authoritative server runs on 127.0.0.1, port 2153
policy.add(policy.suffix(policy.STUB({'127.0.0.1@2153'}), internal_domains))

-- Cache size
cache.size = 100 * MB

policy.add(
  policy.all(
    policy.TLS_FORWARD({
      {'8.8.8.8', hostname='dns.google' },
      {'8.8.4.4', hostname='dns.google' },
      {'1.1.1.1', hostname='cloudflare-dns.com' },
      {'1.0.0.1', hostname='cloudflare-dns.com' },
      {'9.9.9.9', hostname='dns.quad9.net' }
    })
))
```

IP «192.168.5.2» — это локальный IP-адрес сервера FreeBSD, а IP «8.8.8.8,8.8.4.4,1.1.1.1,1.0.0.1,9.9.9.9» — это публичные IP-адреса DNS. Таким образом, резолвер Knot будет «перенаправлять» запросы на публичный DNS-IP. Локальное доменное имя в приведенном выше скрипте — «datainchi.com».

После настройки файла «kresd.conf» перейдите к редактированию файла «/etc/resolv.conf». Введите приведенный ниже скрипт в файл.

```
root@ns3:~ # ee /etc/resolv.conf

domain datainchi.com
nameserver 192.168.5.2
```

## C. Как использовать Knot Resolver

Несмотря на то, что вы настроили файл «kresd.conf», резолвер Knot еще не готов к использованию, он установлен, но не запущен. Чтобы автоматически запустить резолвер Knot, откройте файл «/etc/rc.conf» и введите в него приведенный ниже скрипт.

```
kresd_enable="YES"
kresd_config="/usr/local/etc/knot-resolver/kresd.conf"
kresd_user="kresd"
kresd_group="kresd"
kresd_rundir="/var/run/kresd"

krescachegc_enable="YES"
krescachegc_millis="1000"
```

После этого вы запускаете команду «chown», чтобы предоставить право собственности на файл.

```
root@ns3:~ # chown -R kresd:kresd /usr/local/etc/knot-resolver
```

Перезагрузить (перезапустить) решатель узлов.

```
root@ns3:~ # service kresd restart
root@ns3:~ # service krescachegc restart
```

Теперь ваш разрешитель узлов активен и готов к использованию. Попробуйте провести тестирование с помощью команды «dig».

```
root@ns3:~ # dig google.com

; <<>> DiG 9.18.20 <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 52479
;; flags: qr rd ra; QUERY: 1, ANSWER: 6, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             162     IN      A       142.251.10.113
google.com.             162     IN      A       142.251.10.138
google.com.             162     IN      A       142.251.10.139
google.com.             162     IN      A       142.251.10.100
google.com.             162     IN      A       142.251.10.101
google.com.             162     IN      A       142.251.10.102

;; Query time: 94 msec
;; SERVER: 192.168.5.2#53(192.168.5.2) (UDP)
;; WHEN: Mon Jan 29 16:57:48 WIB 2024
;; MSG SIZE  rcvd: 135
```

Обратите внимание на синий скрипт. Вы успешно запустили резолвер Knot, поскольку на DNS-вызов google.com отвечает локальный IP-адрес вашего сервера FreeBSD. Повторите тест, используя команду ниже.

```
root@ns3:~ # dig oracle.com +trace
root@ns3:~ # dig -x 108.59.161.1
root@ns3:~ # nslookup facebook.com
root@ns3:~ # dig oracle.com +short
root@ns3:~ # dig NS +short unixwinbsd.site
```

В этой статье вы узнали, как установить пакет knot-resolver, настроить его и запустить на сервере FreeBSD. Вы можете изменить файл «kresd.conf», чтобы максимально эффективно использовать приложение Knot resolver. Содержание этой статьи ограничивается только базовой теорией решателя узлов, мы продолжим в следующем обсуждении, чтобы вы могли ощутить преимущества всех функций решателя узлов.



