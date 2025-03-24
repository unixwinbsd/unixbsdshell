---
title: Простой способ создания диаграмм с помощью JAVA Kroki на FreeBSD
date: "2025-02-01 17:01:10 +0100"
id: easy-diagram-with-java-kroki-freebsd
lang: ru
categories:
  - FreeBSD
tags: "WebServer"
excerpt: The Kroki library follows a modular architecture that provides many different modules such as a Java web server that acts as a gateway
keywords: java, freebsd, php, kroki, javascript, apache, tomcat, maven
layout: single
author_profile: true
---

Бесплатная библиотека Java позволяет разработчикам программного обеспечения создавать диаграммы из текстовых описаний. Kroki — это унифицированный API Java с открытым исходным кодом, лицензированный MIT, который позволяет разработчикам программного обеспечения легко создавать диаграммы из текстовых описаний в своих приложениях Java. Kroki — это высокостабильный унифицированный API для всех библиотек диаграмм, который можно использовать где угодно.

Библиотека Kroki следует модульной архитектуре, которая предоставляет множество различных модулей, таких как веб-сервер Java, который действует как шлюз, API Umlet Java для создания диаграмм, Node.js CLI и многое другое. Kroki также поддерживает BlockDiag (BlockDiag, SeqDiag, ActDiag, NwDiag, PacketDiag, RackDiag), BPMN, Bytefield, C4 (с PlantUML), D2, DBML, Ditaa, Erd, Excalidraw, GraphViz, Mermaid, Nomnoml, Pikchr, PlantUML, Structurizr, SvgBob, Symbolator, TikZ, UMLet, Vega, Vega-Lite, WaveDrom, WireViz.<br><br/>
## 1. Функции Kroki
Библиотека Kroki отличается высокой производительностью и скоростью. Вы можете легко взаимодействовать с библиотекой с помощью любого HTTP-клиента. Она предоставляет HTTP API для создания диаграмм из текстовых описаний и может обрабатывать как GET-, так и POST-запросы. Kroki поддерживает функцию кодирования диаграмм и позволяет пользователям использовать алгоритм deflate + base64 с GET-запросом.

У Kroki есть несколько преимуществ, в том числе:

1. Простой
Kroki предоставляет единый API для всех библиотек диаграмм.
2. Быстрый
Создан с использованием современной архитектуры, Kroki обеспечивает отличную производительность.
3. Готов к использованию
Библиотеки диаграмм написаны на разных языках: Haskell, Python, JavaScript, Go, PHP, Java.
4. Бесплатный и с открытым исходным кодом
Весь код доступен на GitHub.
<br><br/>
## 2. Быстрый старт Kroki на FreeBSD
Чтобы запустить Kroki на FreeBSD, сначала нужно установить JAVA. Прочитайте предыдущую статью об установке Java на FreeBSD.

В этой статье мы предполагаем, что вы установили JAVA на FreeBSD, поэтому сразу же приступим к установке библиотеки JAVA, которая будет использоваться для запуска Kroki.

```
root@ns7:~ # pkg install graphviz erd svgbob
```

Вы также можете создать установку Kroki вручную, чтобы настроить ее под свои нужды. Для этого вам нужно вручную установить сервер шлюза Kroki как автономный исполняемый файл jar, установить любые библиотеки диаграмм, которые вы хотите использовать, а затем запустить файл jar сервера шлюза. Подробнее о руководстве по [установке Kroki](https://docs.kroki.io/kroki/setup/install/).

```
mkdir -p root@ns7:~ # mkdir -p ~/kroki-server
root@ns7:~ # cd ~/kroki-server
root@ns7:~/kroki-server #
```

```
root@ns7:~/kroki-server # pwd
/root/kroki-server
```

После того, как мы закончили создание папки Kroki, мы продолжаем загрузку файла Kroki с Github.

```
root@ns7:~/kroki-server # fetch https://github.com/yuzutech/kroki/releases/download/v0.23.0/kroki-standalone-server-v0.23.0.jar

root@ns7:~/kroki-server # file kroki-standalone-server-v0.23.0.jar
```

Запускаем сервер Kroki.

```
root@ns7:~/kroki-server # java -jar kroki-server-v0.16.0.jar
```

Приведенная выше команда откроет веб-сервер Kroki на порту 8080. Вы можете изменить порт в соответствии с вашей системой FreeBSD.

С помощью веб-браузера откройте **http://localhost:8000/.**

Прелесть Kroki в том, что он также предоставляет HTTP API для создания диаграмм, к которым можно получить доступ с помощью таких инструментов, как cURL. Попробуйте Kroki для приятного создания диаграмм!.
