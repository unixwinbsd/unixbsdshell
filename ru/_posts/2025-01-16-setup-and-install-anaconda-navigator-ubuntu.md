---
title: Как установить и использовать Anaconda Navigator
date: "2025-01-16 11:02:00 +0900"
id: setup-and-install-anaconda-navigator-ubuntu
lang: ru
layout: single
author_profile: true
categories:
  - Linux
tags: "UnixShell"
excerpt: "Anaconda — дистрибутив языков программирования Python и R, включающий в себя набор популярных библиотек."
keywords: anaconda, navigator, freebsd, server, openbsd, linux, ubuntu, ipynb
---

Anaconda — дистрибутив языков программирования Python и R, включающий в себя набор популярных библиотек. Основная цель — предоставить последовательный набор тематических модулей, представляющих наибольший интерес для широкого круга пользователей, с разрешением возникающих зависимостей и конфликтов, которых невозможно избежать с помощью одной установки. По состоянию на 2019 год он содержит более 1,5 тысяч модулей.

Главной особенностью этого дистрибутива является собственный менеджер разрешения зависимостей conda с графическим интерфейсом Anaconda Navigator, который позволяет отказаться от стандартного менеджера пакетов (например, pip для Python). Дистрибутив загружается один раз, а вся последующая настройка, включая установку дополнительных модулей, может производиться офлайн. Кроме того, он обеспечивает возможность поддержки нескольких изолированных сред с отдельным разрешением зависимостей версий в каждой среде.

Почему я выбрал Анаконду? Это простое создание изолированных инструментов, интеграция с IDE PyCharm и, как уже упоминалось выше, возможность установки библиотек без доступа к сети. Дистрибутив Anaconda поставляется с инструментом командной строки conda и графическим пользовательским интерфейсом рабочего стола под названием Anaconda Navigator.

При использовании нового навигатора люди всегда задаются вопросом, как он будет работать с имеющимся у них пакетом, а также должен ли он быть совместим с операционной системой, которой вы управляете. Если вы хотите узнать больше об установке, вам необходимо иметь базовые знания о навигаторе и этапах его установки.<br><br/>
## Основные характеристики Anaconda Navigator
Существует множество пакетов, которые можно запустить в Anaconda Navigator. Это зависит от конкретной версии некоторых пакетов. Специалисты по знаниям и специалисты по данным используют несколько версий различных пакетов и сред для разделения этих различных версий. Имеется программа командной строки Anaconda, а также менеджер пакетов и сред.

Это облегчает работу специалистов по обработке данных, поскольку гарантирует, что каждая версия каждого пакета имеет все необходимые зависимости и работает правильно. Это простой способ работы, работающий по принципу «укажи и щелкни». Для этого вам не нужно вводить команды Conda в терминале. Все действия можно выполнять в панели управления Anaconda Navigator: искать пакеты, устанавливать их, запускать и обновлять. Продвинутые пользователи Conda также могут создавать собственные приложения-навигаторы.

Репозиторий Anaconda, известный как облако Anaconda, помогает устанавливать пакеты с открытым исходным кодом один за другим. Также можно создавать собственные пакеты и делиться ими с другими, загружая их в Anaconda Cloud. Уникальные особенности Anaconda включают в себя открытый исходный код процесса установки, простоту использования, высокую производительность за счет использования языков Python и R и многое другое.

Anaconda Navigator содержит самые полезные и популярные приложения, такие как JupyterLab, Jupyter Notebook и Rstudio. Специалисты по обработке данных считают их наиболее важными приложениями. Когда все три доступны на одной платформе, люди перестают искать другие приложения. Три раздела на главной странице очень важны: окружающая среда, обучение и сообщество.<br><br/>
## Как установить Anaconda Navigator
Почти для каждой операционной системы, характерной чертой которой является Anaconda, является отсутствие репозиториев Anaconda по умолчанию. Для установки необходимо сначала скачать Anaconda. Обязательным условием для установки и использования Anaconda Navigator является предварительная установка Anaconda.

В этой статье блогера мы предполагаем, что вы установили Anaconda на свой сервер [Ubuntu](https://ubuntu.com/). Итак, приступим непосредственно к созданию виртуальной среды для Anaconda Navigator. Выполните следующие действия для установки Anaconda Navigator.

```
root@hostname1:~# cd /usr/local/etc
root@hostname1:/usr/local/etc# conda create --name navigatorvirtual
```

После создания папки виртуальной среды для Anaconda Navigator можно приступить к активации виртуальной среды.

```
root@hostname1:/usr/local/etc# conda activate navigatorvirtual
(navigatorvirtual) root@hostname1:/usr/local/etc#
```

С помощью указанной выше команды вы активировали виртуальную рабочую среду для Anaconda Navigator. Следующим шагом будет установка некоторых зависимостей, необходимых для Anaconda Navigator.

```
(navigatorvirtual) root@hostname1:/usr/local/etc# conda install minimal minimalegl offscreen vnc webgl xcb eglfs PyQt5
```

После того, как все установлено, приступайте к установке Anaconda Navigator.

```
(navigatorvirtual) root@hostname1:/usr/local/etc# conda install anaconda::anaconda-navigator
```

При установке Anaconda Navigator, Anaconda автоматически создает 2 канала, а именно: defaults и anaconda. Следующая команда, которую вам нужно выполнить, — активировать Anaconda Navigator.

```
(navigatorvirtual) root@hostname1:/usr/local/etc# source ~/anaconda3/bin/activate root
(base) root@hostname1:/usr/local/etc#
```

Теперь приступим к установке anaconda-client, выполнив следующую команду.

```
(base) root@hostname1:~# conda create --name anaconda-client
(base) root@hostname1:~# conda activate anaconda-client
(anaconda-client) root@hostname1:~# conda install anaconda-client
```

Чтобы проверить, запущен ли anaconda-client, выполните следующую команду.

```
(base) root@hostname1:/usr/local/etc# anaconda login
Using Anaconda API: https://api.anaconda.org
Username: markshevcenko123
iwanse1212's Password: xxxxxxxxxx
login successful
(base) root@hostname1:/usr/local/etc# anaconda whoami
Using Anaconda API: https://api.anaconda.org
Username: iwanse1212
Member since: Fri Oct  4 09:10:37 2024
  +company: Inchimediatama Nusantara
  +description: UNIX BSD EXPLORE
  +location: Indonesia
  +name: markshevcenko
  +url:
  +user_type: user
(base) root@hostname1:/usr/local/etc#
```

Чтобы запустить клиент Anaconda, вам необходимо сначала войти в систему, используя имя пользователя и пароль, которые вы используете для входа в свою учетную запись Anaconda.

После этого мы продолжим с помощью следующей команды, чтобы определить используемые каналы и обновить версию conda.

```
(anaconda-client) root@hostname1:~# conda update -n base -c anaconda conda
(anaconda-client) root@hostname1:~# conda install conda=24.9.1
(anaconda-client) root@hostname1:~# conda install anaconda::anaconda-navigator
```

Последний шаг — запуск Anaconda Navigator.

```
(anaconda-client) root@hostname1:~# anaconda-navigator
```

Профессиональным пользователям [Python](https://www.inchimediatama.org/2024/11/python-pip-pipx-menginstal-dan.html) совершенно неясно, какую консоль Python им следует использовать. Перед созданием Anaconda Navigator программисты писали скрипты Python в терминале и оболочке. Однако в настоящее время в Linux наиболее часто используемыми интерпретаторами Python являются Anaconda Navigator и JupyterLab. В этой статье мы рассмотрим, как установить Anaconda Navigator.

Говорят, что в будущем Python станет лидером среди языков программирования. Если вы программист на Python или хотите войти в мир языков программирования с помощью Python, я надеюсь, что этот пост поможет вам выбрать лучший интерпретатор Python.
