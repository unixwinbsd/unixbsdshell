---
title: "Пошаговое руководство FreeBSD по установке Yii PHP Framework"
date: 2024-05-22 19:00:00+03:00
excerpt: "Преимущества Yii по сравнению с другими фреймворками — высокая скорость и отличная поддержка ООП."
id: installing-yii-php-framework-on-freebsd
lang: ru
layout: single
author_profile: true
categories:
  - FreeBSD
tags: "WebServer"
keywords: yii, freebsd, linux, openbsd, configuration, setup, php, framework
---

Yii — это сокращение от yes, что означает компонентно-ориентированный PHP-фреймворк. Yii обладает высокой производительностью, что делает его очень подходящим для масштабируемых веб-приложений. Yii поставляется с множеством функций, включая MVC, DAO/ActiveRecord, I18N/L10N. Фреймворк также поддерживает AJAX на основе jQuery, управление доступом на основе ролей, скаффолдинг, проверку входных данных, виджеты, события, темы, веб-сервисы и многое другое. Написанный на чистом ООП, Yii прост в использовании, а также очень гибок и расширяем.

Преимущества Yii по сравнению с другими фреймворками — высокая скорость и отличная поддержка ООП. В Yii также есть множество библиотек. С помощью Yii вы сможете легко создавать веб-приложения, отвечающие всем современным стандартам. Встроенные методы — одна из функций, которая может помочь вам значительно сократить объем кода.

Фреймворк — тип фреймворка для разработки веб-приложений. Фреймворки могут помочь упростить и ускорить разработку веб-проектов.

Yii выпускается под лицензией BSD, которая может использоваться в коммерческих целях и встраиваться в проприетарные продукты. Yii имеет отличную документацию и большое сообщество. Большое сообщество разработчиков — это возможность быстро получить помощь и обсудить важные темы.

В этой статье объясняется, как установить и запустить фреймворк Yii на сервере FreeBSD.

## Руководство по установке Yii PHP Framework
Пакеты PKG и порты FreeBSD недоступны в Yii, вам необходимо установить Yii с помощью Composer. Поэтому убедитесь, что на вашем FreeBSD установлен PHP Composer. Чтобы установить Yii на FreeBSD, следуйте инструкциям ниже.

```
root@ns3:~ # cd /usr/local/www
root@ns3:/usr/local/www # git clone https://github.com/yiisoft/yii2.git
```

После загрузки Yii из репозитория Github вы можете установить Yii двумя способами:
- Использование Композитора.
- Загрузите архивы Yii напрямую.

Для большинства людей установка Yii с помощью Composer предпочтительнее, поскольку она позволяет устанавливать новые расширения или выполнять обновления Yii с помощью одной командной строки. В Yii 2, в отличие от Yii 1, после завершения процесса установки вы получите шаблоны фреймворка и приложения. Выполните следующую команду для установки Yii 2 с помощью composer.

```
root@ns3:/usr/local/www # cd yii2
root@ns3:/usr/local/www/yii2 # composer create-project --prefer-dist --stability=dev yiisoft/yii2-app-basic basic
```

## Создать базу данных для Yii
Yii 2 использует базу данных в качестве бэкэнда, ее функция — хранить всю информацию о пользователях Yii. В этой статье мы будем использовать сервер баз данных, который обычно применяется разработчиками, а именно сервер MySQL.

Мы создадим базу данных Yii 2 с использованием MySQL. С помощью PHP база данных MySQL будет подключена к серверу Yii 2. В руководстве ниже будет описан процесс создания базы данных Yii 2.

```
root@ns3:~ # mysql -u root -p
Enter password:
root@localhost [(none)]> CREATE DATABASE yii2basic CHARACTER SET utf8;
root@localhost [(none)]> CREATE USER 'useryii'@'localhost' IDENTIFIED BY 'freebsdyii';
root@localhost [(none)]> GRANT ALL PRIVILEGES ON yii2basic.* TO 'useryii'@'localhost';
root@localhost [(none)]> FLUSH PRIVILEGES;
root@localhost [(none)]> exit;
Bye
root@ns3:~ #
```

После этого откройте каталог /usr/local/www/yii2/basic/config, найдите файл db.php и измените скрипт, как в примере ниже.

```
root@ns3:~ # cd /usr/local/www/yii2/basic/config
root@ns3:/usr/local/www/yii2/basic/config # ee db.php
<?php

return [
    'class' => 'yii\db\Connection',
    'dsn' => 'mysql:host=localhost;dbname=yii2basic',
    'username' => 'root',
    'password' => 'freebsdyii',
    'charset' => 'utf8',

    // Schema cache options (for production environment)
    //'enableSchemaCache' => true,
    //'schemaCacheDuration' => 60,
    //'schemaCache' => 'cache',
];
```

Совместите пароль с паролем базы данных, который вы создали выше. Последний шаг — запуск сервера Yii 2 с помощью следующей команды.

```
root@ns3:/usr/local/www/yii2/basic/config # cd /usr/local/www/yii2/basic
root@ns3:/usr/local/www/yii2/basic # php yii serve 192.168.5.2 --port=8888
```

Откройте веб-браузер Google Chrome и введите «http://192.168.5.2:8888/», посмотрите результат.

Мы предполагаем, что после прочтения этой статьи вы успешно установили Yii 2 на FreeBSD. Кроме того, вы также узнали о Yii PHP Framework и его возможностях. Вы можете изменить каждую функцию в соответствии с потребностями веб-сайта, над которым вы работаете.



