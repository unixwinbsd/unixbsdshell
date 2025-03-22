---
title: Учебное пособие по FreeBSD — настройка Apache mod OpenSSL для HTTPS
date: "2024-07-25 19:00:00 +0300"
id: freebsd-tutorial-setuo-apache-mod-openssl-http
excerpt: "Веб-сервер Apache существует уже много лет, и теперь число его пользователей продолжает расти."
mathjax: true
lang: ru
tags: "apache, freebsd, mod, openbsd, configuration, setup, openssl, http, https, ssl"
keywords: apache, freebsd, mod, openbsd, configuration, setup, openssl, http, https, ssl
---

Веб-сервер Apache существует уже много лет, и теперь число его пользователей продолжает расти. mod_ssl обеспечивает лучшую безопасность веб-сервера Apache и может быть установлен практически на все версии Apache и все операционные системы, такие как FreeBSD, Linux, MacOS и Windows.

Протокол Secure Sockets Layer или часто называемый SSL — это протокол безопасности, который можно разместить между протоколом сетевого уровня TCP/IP и протоколом прикладного уровня HTTP. Мод SSL на Apache обеспечивает безопасную связь между клиентом и сервером. Этот мод будет выполнять аутентификацию и использовать цифровые подписи для целостности и шифрование для конфиденциальности. В настоящее время все еще используются две версии SSL, а именно версия 2 и версия 3.

SSL/TLS использует криптографию с открытым ключом (PKC), известную как асимметричная криптография PKC. Криптография с открытым ключом используется в ситуациях, когда клиент и сервер не имеют общего секрета, например, между браузером и веб-сервером, но оба хотят построить систему безопасности с доверенным каналом для своих коммуникаций.

Криптография с открытым ключом или PKC определяет алгоритм, который использует два ключа, каждый из которых может использоваться для шифрования сообщения. Если один ключ используется для шифрования сообщения, то другой ключ должен использоваться для его расшифровки. Этот процесс выполняется для получения защищенных сообщений путем публикации только одного ключа (открытого ключа) и сохранения другого ключа (закрытого ключа) в секрете.

Открытый ключ может быть зашифрован кем угодно, но прочитать его может только владелец закрытого ключа. Например, Мэри отправляет личное сообщение владельцу пары ключей (например, вашему веб-серверу), затем сообщение от Мэри будет зашифровано с использованием открытого ключа, опубликованного вашим сервером. Только вы можете расшифровать его, используя соответствующий закрытый ключ.

Руководство в этой статье поможет вам включить SSL mod для веб-сайтов, обслуживаемых веб-серверами Apache. Содержание этой статьи сосредоточено только на создании SSL на Apache с помощью приложения OpenSSL.

## 1. Установите OpenSSL

OpenSSL — это утилита для протоколов Transport Layer Security (TLS) и Secure Sockets Layer (SSL). OpenSSL — это бесплатное приложение и наиболее широко используемая криптографическая библиотека. Основная цель создания приложений — защита соединений на серверах и в вашем программном обеспечении.

Процесс установки OpenSSL на FreeBSD довольно прост, нет никаких настроек Start up rc.d, но настройка и реализация немного сложны и иногда запутанны. Пожалуйста, используйте шаги, описанные в этой статье.

```
root@ns3:~ # pkg install perl5
```

После установки Perl вы можете сразу установить OpenSSL. Мы рекомендуем использовать системные порты FreeBSD, чтобы процесс установки был более совершенным.

```
root@ns3:~ # cd /usr/ports/security/openssl
root@ns3:/usr/ports/security/openssl # make config
root@ns3:/usr/ports/security/openssl # make install clean
```

На FreeBSD есть много приложений, похожих на OpenSSL, например ca_root_nss, LibreSSL и другие. Чтобы FreeBSD по умолчанию использовала безопасность OpenSSL для шифрования и дешифрования, в /etc/make.conf введите следующую команду.

```
DEFAULT_VERSIONS+=ssl=openssl
```

## 2. Создайте файл server.crt и server.key
Первый шаг, который вам нужно сделать, чтобы Apache мог подключиться к OpenSSL, — создать файл server.key и server.crt. Для этого мы сначала создаем папку SSL.

```
root@ns3:~ # cd /usr/local/etc/apache24
root@ns3:/usr/local/etc/apache24 # mkdir -p ssl
root@ns3:/usr/local/etc/apache24 # cd ssl
```

Сгенерируйте server.key с помощью openssl.

```
root@ns3:/usr/local/etc/apache24/ssl # openssl genrsa -des3 -out server.key 1024
Generating RSA private key, 1024 bit long modulus (2 primes)
..+++++
.............................................+++++
e is 65537 (0x010001)
Enter pass phrase for server.key:
Verifying - Enter pass phrase for server.key:
```

Приведенная выше команда создаст файл server.key и запросит пароль. Обязательно запомните этот пароль. Он вам понадобится при следующем запуске Apache.

Далее создайте файл запроса сертификата, а именно "server.csr", процесс создания этого файла будет использовать файл server.key, указанный выше.

```
root@ns3:/usr/local/etc/apache24/ssl # openssl req -new -key server.key -out server.csr
```

Последний шаг — создание самоподписанного SSL-сертификата (server.crt) с использованием файлов server.key и server.csr, указанных выше.

```
root@ns3:/usr/local/etc/apache24/ssl # openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

## 3. Включить режим SSL
Приведенные ниже инструкции помогут вам включить режим SSL на Apache. Цель состоит в том, чтобы OpenSSL и Apache могли взаимодействовать друг с другом для защиты вашего веб-сервера.

Почти во всех операционных системах файл конфигурации Apache называется "httpd.conf", единственное отличие — расположение файла. В FreeBSD этот файл находится в каталоге "/usr/local/etc/apache24", откройте файл и активируйте скрипт SSL в файле, как в примере ниже.

```
LoadModule ssl_module libexec/apache24/mod_ssl.so
LoadModule socache_shmcb_module libexec/apache24/mod_socache_shmcb.so
Include etc/apache24/extra/httpd-ssl.conf
```

В скрипте выше мы активируем файл httpd-ssl.conf. Откройте файл и удалите все содержимое скрипта, затем вставьте скрипт ниже в файл "/usr/local/etc/apache24/extra/httpd-ssl.conf".

```
Listen 443

SSLCipherSuite HIGH:MEDIUM:!MD5:!RC4:!3DES
SSLProxyCipherSuite HIGH:MEDIUM:!MD5:!RC4:!3DES

SSLHonorCipherOrder on 

SSLProtocol all -SSLv3
SSLProxyProtocol all -SSLv3

SSLPassPhraseDialog  builtin
SSLSessionCache        "shmcb:/var/run/ssl_scache(512000)"
SSLSessionCacheTimeout  300

<VirtualHost _default_:443>
DocumentRoot "/usr/local/www/apache24/data"
ServerName www.datainchi.com:443
ServerAdmin datainchi@gmail.com
ErrorLog "/var/log/httpd-error.log"
TransferLog "/var/log/httpd-access.log"

SSLEngine on

SSLCertificateFile "/usr/local/etc/apache24/ssl/server.crt"
SSLCertificateKeyFile "/usr/local/etc/apache24/ssl/server.key"

<FilesMatch "\.(cgi|shtml|phtml|php)$">
    SSLOptions +StdEnvVars
</FilesMatch>
<Directory "/usr/local/www/apache24/cgi-bin">
    SSLOptions +StdEnvVars
</Directory>

BrowserMatch "MSIE [2-5]" nokeepalive ssl-unclean-shutdown downgrade-1.0 force-response-1.0
CustomLog "/var/log/httpd-ssl_request.log" "%t %h %{SSL_PROTOCOL}x %{SSL_CIPHER}x \"%r\" %b"

</VirtualHost>
```

Обратите внимание на синий скрипт, скрипт — это файл OpenSSL, который мы создали выше. Чтобы протестировать все указанные выше конфигурации, перезапустите сервер Apache.

```
root@ns3:~ # service apache24 restart
Performing sanity check on apache24 configuration:
Syntax OK
Stopping apache24.
Waiting for PIDS: 36862.
Performing sanity check on apache24 configuration:
Syntax OK
Starting apache24.
Apache/2.4.58 mod_ssl (Pass Phrase Dialog)
Some of your private key files are encrypted for security reasons.
In order to read them you have to provide the pass phrases.

Private key www.datainchi.com:443:0 (/usr/local/etc/apache24/ssl/server.key)
Enter pass phrase:

OK: Pass Phrase Dialog successful.
```

Вам будет предложено ввести пароль, введите пароль из файла OpenSSL, который вы создали выше.

Следующий тест — открыть веб-браузер «Google Chrome» и ввести команду «https://192.168.5.2/».

В этом руководстве показана базовая установка и использование mod_ssl на сервере Apache с OpenSSL. Несмотря на то, что команда очень проста, ее способность защищать веб-серверы очень надежна и доказана, поскольку многие люди используют OpenSSL в качестве системы безопасности SSL на серверах Apache.
