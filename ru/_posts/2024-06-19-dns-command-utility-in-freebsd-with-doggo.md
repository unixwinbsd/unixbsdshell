---
title: "Утилита управления DNS на FreeBSD с Doggo"
date: 2024-06-19 19:00:00+03:00
excerpt: "Doggo — современный DNS-клиент командной строки, написанный на Golang. Поэтому не будет ошибкой назвать его «dog + go = doggo»."
id: dns-command-utility-in-freebsd-with-doggo
lang: ru
layout: single
author_profile: true
categories:
  - FreeBSD
tags: "DNSServer"
keywords: doggo, freebsd, dig, openbsd, configuration, setup, nslookup, command
---

Doggo — это современная утилита командной строки для поиска DNS, похожая на dig, с цветным выводом, поддержкой протоколов DNS-over-TLS и DNS-over-HTTPS и многим другим. Doggo доступен практически во всех операционных системах UNIX, Linux, macOS, а также Microsoft Windows. Благодаря красочному выводу и поддержке DNSCrypt, DOT и DOT Doggo очень удобен в использовании.

Doggo — современный DNS-клиент командной строки, написанный на Golang. Поэтому не будет ошибкой назвать его «dog + go = doggo». Утилита Doggo отображает информацию лаконично и аккуратно. Утилита doggo очень похожа на dig: doggo выполняет поиск DNS и отображает ответы, возвращенные запрошенными серверами имен, что полезно для устранения неполадок DNS.

В этой статье мы обсудим, как установить и использовать doggo в системе FreeBSD.

## Особенности Doggo
1. Цветной вывод
2. Поддерживает формат json
3. Поддерживает несколько транспортных протоколов
DNS через HTTPS (DoH)
DNS через TLS (DoT)
DNS через QUIC (DoQ)
DNS через TCP
DNS через UDP
DNS через DNSCrypt
4. Поддерживает ndots и конфигурацию поиска из resolv.conf или аргументов командной строки.
5. Поддерживает IP4 и IP6.
6. Поддерживает несколько резолверов одновременно.
7. Доступно как веб-инструмент, посетите: https://doggo.mrkaran.dev.
8. Можно использовать с zsh и fish shell.
9. Возможность выполнять обратный DNS-поиск.

## Процесс установки Doggo
Чтобы установить Doggo на FreeBSD, необходимо использовать систему портов FreeBSD. Вот пример скрипта установки doggo.

```
root@ns1:~ # cd /usr/ports/dns/doggo
root@ns1:/usr/ports/dns/doggo # make install clean
```

## Как использовать Doggo
### а. Выполните поиск DNS на google.com

```
root@ns1:~ # doggo google.com
```

### Запрос записи MX для google.com с использованием Resolver 8.8.8.8

```
root@ns1:~ # doggo MX google.com @8.8.8.8
```
```
root@ns1:~ # doggo -t MX -n 1.1.1.1 google.com
NAME       	TYPE	CLASS	TTL 	ADDRESS            	        NAMESERVER 
google.com.	MX  	IN   	        300s	10 smtp.google.com.	1.1.1.1:53	
google.com.	MX  	IN   	        300s	10 smtp.google.com.	1.0.0.1:53
```

### Отображение записей DNS запроса для archive.org с использованием DoH-резолвера Cloudflare

```
root@ns1:~ # doggo archive.org @https://cloudflare-dns.com/dns-query 
NAME        	TYPE	CLASS	TTL 	ADDRESS      	NAMESERVER                           
archive.org.	A   	IN   	216s	207.241.224.2	https://cloudflare-dns.com/dns-query
```

### Запрос данных DNS для unixwinbsd.blogspot.com с выводом JSON

```
root@ns1:~ # doggo unixwinbsd.blogspot.com --json
[
    {
        "answers": [
            {
                "name": "unixwinbsd.blogspot.com.",
                "type": "CNAME",
                "class": "IN",
                "ttl": "300s",
                "address": "blogspot.l.googleusercontent.com.",
                "status": "",
                "rtt": "19ms",
                "nameserver": "1.1.1.1:53"
            },
            {
                "name": "blogspot.l.googleusercontent.com.",
                "type": "A",
                "class": "IN",
                "ttl": "263s",
                "address": "74.125.200.132",
                "status": "",
                "rtt": "19ms",
                "nameserver": "1.1.1.1:53"
            }
        ],
        "authorities": null,
        "questions": [
            {
                "name": "unixwinbsd.blogspot.com.",
                "type": "A",
                "class": "IN"
            }
        ]
    },
    {
        "answers": [
            {
                "name": "unixwinbsd.blogspot.com.",
                "type": "CNAME",
                "class": "IN",
                "ttl": "300s",
                "address": "blogspot.l.googleusercontent.com.",
                "status": "",
                "rtt": "0ms",
                "nameserver": "1.0.0.1:53"
            },
            {
                "name": "blogspot.l.googleusercontent.com.",
                "type": "A",
                "class": "IN",
                "ttl": "263s",
                "address": "74.125.200.132",
                "status": "",
                "rtt": "0ms",
                "nameserver": "1.0.0.1:53"
            }
        ],
        "authorities": null,
        "questions": [
            {
                "name": "unixwinbsd.blogspot.com.",
                "type": "A",
                "class": "IN"
            }
        ]
    }
```

### Запрос данных DNS для google.com и отображение RTT

```
root@ns1:~ # doggo google.com --time
NAME                       	        TYPE 	    CLASS	TTL   	ADDRESS                    	        NAMESERVER	TIME TAKEN 
google.com.                	        CNAME	    IN   	        265s  	forcesafesearch.google.com.	1.1.1.1:53	        8ms       	
forcesafesearch.google.com.	A    	            IN   	        81816s	216.239.38.120             	        1.1.1.1:53	        8ms       	
google.com.                	        CNAME	    IN   	        265s  	forcesafesearch.google.com.	1.0.0.1:53	        0ms       	    
forcesafesearch.google.com.	A    	            IN       	        81816s	216.239.38.120             	        1.0.0.1:53	        0ms
```

### Запросить записи A, NS и MX для домена duckduckgo.com

```
root@ns1:~ # doggo google.com A NS MX
NAME                       	        TYPE 	CLASS	TTL    	ADDRESS                    	        NAMESERVER 
google.com.                	        CNAME	IN   	        145s   	forcesafesearch.google.com.	1.1.1.1:53	
forcesafesearch.google.com.	A    	        IN   	        66288s 	216.239.38.120             	        1.1.1.1:53	
google.com.                	        CNAME	IN   	        145s   	forcesafesearch.google.com.	1.0.0.1:53	
forcesafesearch.google.com.	A    	        IN   	        66288s 	216.239.38.120             	        1.0.0.1:53	
google.com.                	        NS   	IN   	        177459s	ns2.google.com.            	        1.1.1.1:53	
google.com.                	        NS   	IN   	        177459s	ns3.google.com.            	        1.1.1.1:53	
google.com.                	        NS   	IN   	        177459s	ns4.google.com.            	        1.1.1.1:53	
google.com.                	        NS   	IN   	        177459s	ns1.google.com.            	        1.1.1.1:53	
google.com.                	        NS   	IN   	        177484s	ns4.google.com.            	        1.0.0.1:53	
google.com.                	        NS   	IN   	        177484s	ns2.google.com.            	        1.0.0.1:53	
google.com.                	        NS   	IN   	        177484s	ns3.google.com.            	        1.0.0.1:53	
google.com.                	        NS   	IN   	        177484s	ns1.google.com.            	        1.0.0.1:53	
google.com.                	        MX   	IN   	        300s   	10 smtp.google.com.        	1.1.1.1:53	
google.com.                	        MX   	IN   	        300s   	10 smtp.google.com.        	1.0.0.1:53
```

### Отправка запроса DOT на порт 853

```
root@ns1:~ # doggo google.com @tls://@1.1.1.1
NAME       	TYPE	CLASS	TTL 	ADDRESS       	NAMESERVER  
google.com.	A   	        IN   	        181s	142.251.12.138	1.1.1.1:853	
google.com.	A   	        IN   	        181s	142.251.12.139	1.1.1.1:853	
google.com.	A   	        IN   	        181s	142.251.12.102	1.1.1.1:853	
google.com.	A   	        IN   	        181s	142.251.12.101	1.1.1.1:853	
google.com.	A   	        IN           	181s	142.251.12.100	1.1.1.1:853	
google.com.	A   	        IN   	        181s	142.251.12.113	1.1.1.1:853
```

Хотя Doggo и менее популярен, чем Dig, его существование стало серьезной проблемой для системных администраторов, желающих заменить Dig. Благодаря своим полным характеристикам и немонотонному внешнему виду Doggo готов конкурировать с Dig.
