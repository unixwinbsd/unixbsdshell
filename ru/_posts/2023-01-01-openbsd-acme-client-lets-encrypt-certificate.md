---
title: Как создать сертификат Lets Encrypt с помощью клиента Acme OpenBSD
date: "2023-01-01 09:00:00 +0300"
id: openbsd-acme-client-lets-encrypt-certificate
lang: ru
layout: single
categories:
  - OpenBSD
excerpt: "Let's Encrypt использует протокол ACME для проверки того, что вы используете определенное доменное имя, и выдает сертификат для этого доменного имени."
tags: "acme, freebsd, client, openbsd, configuration, setup, certificate, openssl, latenscrypt"
keywords: acme, freebsd, client, openbsd, configuration, setup, certificate, openssl, latenscrypt
---

В этой статье мы объясним, как использовать клиент acme для OpenBSD. Acme-client — клиент для стандартной среды автоматизированного управления сертификатами OpenBSD (ACME). Данное программное обеспечение устанавливается во время установки ОС.

Let's Encrypt использует протокол ACME для проверки того, что вы используете определенное доменное имя, и выдает сертификат для этого доменного имени. Чтобы получить сертификат Let's Encrypt, необходимо выбрать клиентское программное обеспечение ACME.

Основной файл конфигурации клиента acme называется acme-client.conf, вы можете редактировать этот файл по мере необходимости. Редактировать файл acme-client.conf не так уж и сложно. Редактируя файл, вы автоматически получите сертификаты от Let's Encrypt и других центров сертификации.
Базовое использование выглядит так:

```
# nvim /etc/acme-client.conf

# # Obtaining a certificate or renewing (for renewal)
# acme-client <domain>

# # Disable (revoke) a certificate
# acme-client -r <domain>
```

# Технические характеристики системы
- OS: OpenBSD 7.5
- ACME client: OpenBSD acme-client
- Certificate authority: Let's Encrypt
- Web server: OpenBSD httpd

## 1. О CertBot
Кстати, помимо acme есть еще один популярный вариант — CertBot. Этот очень популярный клиент был выпущен Electronic Frontier Foundation (EFF). Certbot взаимодействует с центром сертификации Let’s Encrypt для получения и установки сертификатов SSL/TLS.

В OpenBSD CertBot очень легко установить с помощью команды pkg_add certbot. Однако если вы используете это программное обеспечение в OpenBSD, вы можете столкнуться с проблемами разрешений при обновлении сертификатов.

Certbot — это бесплатное программное обеспечение с открытым исходным кодом для автоматического развертывания сертификатов Let’s Encrypt на веб-сайтах для включения HTTPS. Let’s Encrypt — это центр сертификации, который предоставляет бесплатные сертификаты SSL/TLS, а Certbot автоматизирует процесс получения, установки и продления этих сертификатов.

### а. Как использовать
Первый шаг, который вам необходимо сделать, — это подготовить файл acme-client.conf. Пакеты OpenBSD предоставляют официальные файлы конфигурации, примеры скриптов которых можно просмотреть с помощью команды «cat».

```
$ cat /etc/examples/acme-client.conf
```

Команда «cat» отобразит полный скрипт /etc/examples/acme-client.conf, как показано ниже.

```
# $OpenBSD: acme-client.conf,v 1.4 2020/09/17 09:13:06 florian Exp $
#
authority letsencrypt {
    api url "https://acme-v02.api.letsencrypt.org/directory"
    account key "/etc/acme/letsencrypt-privkey.pem"
}

authority letsencrypt-staging {
    api url "https://acme-staging-v02.api.letsencrypt.org/directory"
    account key "/etc/acme/letsencrypt-staging-privkey.pem"
}

authority buypass {
    api url "https://api.buypass.com/acme/directory"
    account key "/etc/acme/buypass-privkey.pem"
    contact "mailto:me@example.com"
}

authority buypass-test {
    api url "https://api.test4.buypass.no/acme/directory"
    account key "/etc/acme/buypass-test-privkey.pem"
    contact "mailto:me@example.com"
}

domain example.com {
    alternative names { secure.example.com }
    domain key "/etc/ssl/private/example.com.key"
    domain full chain certificate "/etc/ssl/example.com.fullchain.pem"
    sign with letsencrypt
}
```

Скопируйте файлы и переместите их в каталог «/etc».

```
$ # In the initial state, nothing will be displayed even if you execute the following (aside)
$ ls /etc/acme-client*

$ doas cp -p /etc/examples/acme-client.conf /etc/
```

После перемещения отредактируйте файл в соответствии с требованиями вашей системы.

```
$ doas nvim /etc/acme-client.conf
```

Вот пример скрипта acme-client.conf после редактирования.

```
- domain example.com {
-   alternative names { secure.example.com }
-   domain key "/etc/ssl/private/example.com.key"
-   domain full chain certificate "/etc/ssl/example.com.fullchain.pem"
-   sign with letsencrypt
- }
+ domain your.domain {    
+   #alternative names { secure.example.com }    
+   domain key "/etc/ssl/private/your.domain.key"    
+   domain full chain certificate "/etc/ssl/your.domain.fullchain.pem"    
+   sign with letsencrypt    
+ }
```

Теперь ваш letsencrypt готов к использованию для вашего доменного имени.

После этого откройте веб-сервер OpenBSD httpd и убедитесь, что порт 80 включен. Проверьте файл httpd.conf, чтобы убедиться, что порт 80 активен.

```
server "your.domain" {
    listen on egress port 80
    # ...
}
```

Кстати, пример файла httpd.conf вы можете найти в /etc/examples/. Затем выполните команду, указанную ниже.

```
$ doas rcctl start -f httpd
$ doas rcctl restart httpd
```

Чтобы получить сертификат letsencrypt, просто выполните однострочную команду ниже, letsencrypt будет автоматически активирован на сервере httpd.

```
$ doas acme-client -v your.domain
```

Здесь -v используется для длинного вывода и является необязательным. Есть и другие варианты использования. Обновления можно сделать следующим образом.

```
$ doas acme-client your.domain
```

Вы также можете отозвать сертификат.

```
$ doas acme-client -r your.domain
```

Вот пример скрипта файла /etc/httpd.conf.

```
chroot "/var/www"
ext_addr="*"

prefork 2

server "www.example.com" {
    listen on $ext_addr tls port 443
    alias "example.com"
    root "/htdocs/public"
    tls {
        certificate "/etc/ssl/www.example.com.pem"
        key "/etc/ssl/private/www.example.com.key"
        ticket lifetime default
        ciphers "secure"
    }

    hsts max-age 16000000
    hsts preload
    hsts subdomains

    location "/.well-known/acme-challenge/*" {
        root "/acme"
        request strip 2
    }

}

server "www.example.com" {
    listen on $ext_addr port 80
    alias "example.com"
    block return 301 "https://www.example.com$REQUEST_URI"
    location "/.well-known/acme-challenge/*" {
        root "/acme"
        request strip 2
    }

}
```

Последний шаг — инициализация сертификата с помощью команды ниже.

```
$ acme-client -ADv www.example.com
```

Обратите внимание, что сертификаты Let's Encrypt действительны только в течение 90 дней. Поэтому сертификат необходимо периодически продлевать. Одним из распространенных способов является использование cron.



