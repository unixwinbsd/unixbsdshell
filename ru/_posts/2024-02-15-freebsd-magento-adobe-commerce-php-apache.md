---
title: Настройка FreeBSD Magento Adobe Commerce с PHP Composer и Apache
date: "2024-02-15 19:00:00 +0300"
id: freebsd-magento-adobe-commerce-php-apache
lang: ru
layout: single
author_profile: true
categories:
  - FreeBSD
excerpt: "Magento, написанная на PHP и разработанная Adobe, представляет собой платформу электронной коммерции с открытым исходным кодом для управления интернет-магазинами."
tags: "SysAdmin"
keywords: adobe, freebsd, magento, openbsd, configuration, setup, server, commerce, client
---

Magento — это приложение CMS Framework, предназначенное для запуска крупномасштабных и сложных проектов электронной коммерции. Основная цель — помочь владельцам интернет-магазинов с сотнями и тысячами товаров. Простота управления тысячами продуктов делает его очень популярным среди крупных предпринимателей.

Magento, написанная на PHP и разработанная Adobe, представляет собой платформу электронной коммерции с открытым исходным кодом для управления интернет-магазинами. Функции Magento предоставляют инструменты для создания и управления интернет-магазинами и платформами для онлайн-торговли. Magento обеспечивает более плавную навигацию, лучшую конверсию и улучшенный пользовательский опыт.

Magento обеспечивает гибкую настройку, масштабируемость и расширяемость для различных типов онлайн-бизнеса. Платформа Magento CMS также поддерживает множество функций, включая управление продуктами, заказы, платежи, доставку и маркетинговые инструменты для улучшения качества обслуживания клиентов. Это позволяет компаниям создавать и управлять эффективными интернет-магазинами с различными функциями и возможностями, которые могут помочь владельцам магазинов развивать свой бизнес.

Magento, также известная как Adobe Commerce, — это решение для электронной коммерции, предназначенное для крупных интернет-торговцев, разработанное с учетом того, что разработчикам необходимо самостоятельно разрабатывать интернет-магазины. Magento обеспечивает простое управление товарными запасами и продажами для лучшего управления вашим интернет-магазином. Этот интернет-магазин также предоставляет конструктор страниц, поэтому вы можете легко создавать и оформлять веб-страницы в соответствии со своими пожеланиями.

В этой статье мы изучим процесс установки и настройки Adobe Magento, чтобы вы могли отображать платформу Magento на экране Google Chrome.

# Технические характеристики системы

- OS: FreeBSD 13.3
- Hostname: ns3
- IP FreeBSD: 192.168.5.2
- Database server: MySQL8
- Web server: Apache24
- PHP: PHP82, PHP Composer
- PHP FPM
- Elasticsearch: elasticsearch8
- Dependencies: php82-opcache php82-session php82-pecl-APCu php82-bcmath php82-bz2 php82-ctype php82-curl php82-dom php82-exif php82-fileinfo php82-filter php82-gd php82-gmp php82-iconv php82-intl php82-ldap php82-mbstring php82-pecl-msgpack php82-mysqli php82-pcntl php82-pdo php82-phar php82-posix php82-simplexml php82-sodium php82-sysvsem php82-tokenizer php82-xml php82-xmlwriter php82-zip php82-zlib php82-pecl-memcache php82-pecl-memcached php82-pdo_mysql php82-xmlreader php82-xsl php82-soap

## Установить зависимости
Adobe Magento имеет множество PHP-зависимостей. При первой настройке Magento необходимо установить зависимости. Потому что если вы не установите зависимости, не ожидайте, что сможете нормально запустить Magento. Все зависимости написаны на PHP, что очень полезно для Magento, поскольку позволяет подключать веб-сайт к базам данных и веб-серверам, таким как Apache.

Чтобы упростить задачу, вы можете установить зависимости Magento с помощью пакета FreeBSD PKG. Вот как установить зависимости Magento с помощью PKG.

```
root@ns3:/usr/local/www # pkg install php82-opcache php82-session php82-pecl-APCu php82-bcmath php82-bz2 php82-ctype php82-curl php82-dom php82-exif php82-fileinfo php82-filter php82-gd php82-gmp php82-iconv php82-intl php82-ldap php82-mbstring php82-pecl-msgpack php82-mysqli php82-pcntl php82-pdo php82-phar php82-posix php82-simplexml php82-sodium php82-sysvsem php82-tokenizer php82-xml php82-xmlwriter php82-zip php82-zlib php82-pecl-memcache php82-pecl-memcached php82-pdo_mysql php82-xmlreader php82-xsl php82-soap
```

## Создать базу данных для Adobe Magento
В этой статье мы предполагаем, что все перечисленные выше системные спецификации установлены на сервере FreeBSD. Поэтому мы не будем обсуждать вышеуказанные приложения. Давайте перейдем непосредственно к основной теме статьи.

Создание базы данных является важной частью настройки Magento. Все ваши данные, такие как название продукта и данные о клиентах, будут храниться в базе данных. В этой статье мы будем использовать сервер MySQL для создания базы данных Magento.

Приведенные ниже команды помогут вам создать базу данных Maento с сервером MySQL.

```
root@ns3:~ # mysql -u root -p
root@localhost [(none)]> CREATE DATABASE magento CHARACTER SET utf8;
root@localhost [(none)]> CREATE USER 'usermagento'@'localhost' IDENTIFIED BY 'router12345';
root@localhost [(none)]> GRANT ALL PRIVILEGES ON magento.* TO 'usermagento'@'localhost';
root@localhost [(none)]> FLUSH PRIVILEGES;
root@localhost [(none)]> exit;
root@ns3:~ #
```

## Создание открытых и закрытых ключей Adobe Magento
Перед началом процесса установки Magento необходимо создать открытый и закрытый ключ Magento. Этот ключ используется для аутентификации доступа к репозиторию repo.magento.com. Потому что каждый раз при доступе к repo.magento.com требуются как открытый, так и закрытый ключи. Этот ключ используется для заполнения:
Username = public key
Password = private key

Получить или сгенерировать ключ можно, зайдя на сайт торговой площадки Magento.
- Первый вход на https://marketplace.magento.com/
- Если у вас нет учетной записи Magento, создайте новую учетную запись по адресу https://account.magento.com/applications/customer/login/
- Если у вас уже есть учетная запись Magento, войдите в систему, используя свои учетные данные Magento.
- После успешного входа откройте меню «Мой профиль».
- Нажмите «Ключ доступа».
- Выберите меню «Magento2».
- Чтобы создать новый ключ, нажмите меню «Создать новый ключ доступа».

## Как установить Adobe Magento
В системах FreeBSD репозиторий Magento недоступен в PKG или портах. При установке Magento и модулей Magento активно используются модули Composer. Поэтому перед установкой Magento необходимо сначала установить Composer. Еще одной важной деталью при использовании Composer является то, что переменная memory_limit (PHP) должна быть установлена ​​на 4 ГБ. При достижении предела в 2 ГБ Composer может вызвать сбой PHP с фатальной ошибкой в ​​процессе установки.

Первым шагом будет создание рабочего каталога для Magento, мы поместим этот каталог в блок сервера Apache, а именно /usr/local/www.

```
root@ns3:~ # mkdir -p /usr/local/www/magento
```

После этого создайте проект Magento с помощью Composer.

```
root@ns3:~ # cd /usr/local/www
root@ns3:/usr/local/www # composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition magento
```

При запуске указанной выше команды Composer запросит имя пользователя и пароль. Используйте пароль и пользователя, созданные вами выше. После этого установите Magento с помощью команды ниже.

```
root@ns3:/usr/local/www # cd magento
root@ns3:/usr/local/www/magento # composer install --ignore-platform-req=ext-sockets
```

Также установите сокеты для PHP.

```
root@ns3:/usr/local/www/magento # pkg install php82-sockets
```

Также выполните команду ниже, чтобы продолжить установку Magento.

```
root@ns3:/usr/local/www/magento/bin # ./magento setup:install --base-url=http://192.168.5.2/magento --db-host=localhost --db-name=magento --db-user=usermagento --db-password=router12345 --admin-firstname=john --admin-lastname=doe --admin-email=datainchi@gmail.com --admin-user=admin --admin-password=admin123 --language=en_US --currency=USD --timezone=America/New_York --use-rewrites=1 --search-engine=elasticsearch8 --elasticsearch-host=http://192.168.5.2 --elasticsearch-port=9200
```

В процессе установки Magento на FreeBSD следует обратить внимание на приложение Elasticsearch. Перед [установкой Magento](https://www.inchimediatama.org/2025/02/freebsd-elasticsearch-cara-mengaktifkan.html) вам необходимо установить это приложение.

После этого откройте файл elasticsearch.yml и отредактируйте скрипт, как в примере ниже.

```
root@ns3:~ # ee /usr/local/etc/elasticsearch/elasticsearch.yml
cluster.name: magento
node.name: node-1
path.data: /var/db/elasticsearch
path.logs: /var/run/elasticsearch
bootstrap.memory_lock: true
network.host: 192.168.5.2
http.port: 9200
discovery.seed_hosts: ["127.0.0.1", "[::1]"]
discovery.type: single-node
xpack.ml.enabled: false
xpack.security.enabled: false
xpack.security.enrollment.enabled: false
```

Начиная с версии Magento 2.2, для управления файлами журналов был добавлен файл crontab. В FreeBSD вы можете управлять файлами crontab в /etc/crontab. Выполните команду ниже, чтобы установить файл crontab.

```
root@ns3:~ # cd /usr/local/www/magento/bin
root@ns3:/usr/local/www/magento/bin # ./magento cron:install --force
Crontab has been generated and saved
```

Затем в файл /etc/crontab добавьте скрипт, указанный ниже.

```
root@ns3:/usr/local/www/magento/bin # ee /etc/crontab
* * * * * /usr/local/bin/php /usr/local/www/magento/bin/magento cron:run 2>&1 | grep -v "Ran jobs by schedule" >> /usr/local/www/magento/var/log/magento.cron.log
```

Далее выполните команды ниже, чтобы обновить базу данных, развернуть файлы статического представления и отключить двухфакторную аутентификацию.

```
root@ns3:/usr/local/www/magento/bin # ./magento indexer:reindex && ./magento se:up && ./magento se:s:d -f && ./magento c:f && ./magento module:disable Magento_TwoFactorAuth
```

Затем вы добавляете права доступа и владельца к файлам и папкам Magento.

```
root@ns3:~ # chown -R www:www /usr/local/www/magento/
root@ns3:~ # chmod -R 775 /usr/local/www/magento
```

## Создать Apache Vhost
В качестве интерфейса Magento мы будем использовать Apache24 как веб-сервер. С помощью Apache Magento можно отображать и настраивать через Google Chrome. Чтобы подключить Magento к Apache, вам необходимо включить файл виртуального хоста Apache. В файле httpd.conf включите файл vhost, удалив знак «#», см. пример ниже.

```
root@ns3:~ # cd /usr/local/etc/apache24
root@ns3:/usr/local/etc/apache24 # ee httpd.conf
#Include etc/apache24/extra/httpd-vhosts.conf
Include etc/apache24/extra/httpd-vhosts.conf
```

После этого удалите все скрипты /usr/local/etc/apache24/extra/vhost.conf и замените их скриптами, приведенными ниже (настройте под свою серверную систему FreeBSD).

```
root@ns3:/usr/local/etc/apache24 # cd extra
root@ns3:/usr/local/etc/apache24/extra # ee httpd-vhosts.conf
<VirtualHost *:80>
    ServerAdmin datainchi@gmail.com
    DocumentRoot "/usr/local/www/magento/pub"
    ServerName datainchi.com
    ServerAlias www.datainchi.com
SetEnv MAGE_RUN_CODE "base"
    SetEnv MAGE_RUN_TYPE "website"

  <Directory /usr/local/www/magento/pub>
       Options Indexes FollowSymLinks MultiViews
       AllowOverride All
       Require all granted
  </Directory>

    ErrorLog "/var/log/dummy-host.example.com-error_log"
    CustomLog "/var/log/dummy-host.example.com-access_log" common
</VirtualHost>
```

Следующий шаг — перезапуск Apache24.

```
root@ns3:~ # service apache24 restart
```

Последний шаг — проверка, для чего откройте Google Chrome и введите «http://192.168.5.2/magento/setup/» или «http://192.168.5.2/magento/pub/». Просмотрите результаты в Google Chrome. Если ваши настройки верны, появится меню конфигурации Magento.

Если вы хотите использовать Magento для открытия интернет-магазина, полное и понятное руководство по установке и настройке облегчит этот процесс. В этом руководстве показано, как вручную настроить Magento на сервере FreeBSD. Однако существующий скрипт можно использовать на серверах Linux, таких как Ubuntu. Мы представляем все содержание этой статьи в полном объеме и можем помочь тем из вас, кто хочет создать интернет-магазин на Magento.






