---
title: "Настройка FreeBSD Lighttpd CGI BIN для программ Perl"
date: 2024-11-28 19:00:00+03:00
excerpt: "Первые программы CGI были написаны на языке C и их нужно было компилировать в исполняемые двоичные файлы."
id: setup-freebsd-lighttpd-cgi-bin-perl
lang: ru
layout: single
author_profile: true
categories:
  - FreeBSD
tags: "WebServer"
keywords: cgi, freebsd, bin, openbsd, configuration, httpd, openssl, http, https, ssl
---

Приложения CGI могут читать и писать на любом компьютерном языке, который является производным от STDIN и выводится в STDOUT, независимо от того, интерпретируется ли программа оболочкой Unix и скомпилирована ли она на C или C++, или на их комбинации, например, Perl. Первые программы CGI были написаны на языке C и их нужно было компилировать в исполняемые двоичные файлы. Поэтому каталог, в котором выполняется скомпилированная CGI-программа, называется cgi-bin, а каталог исходного файла — cgi-src. Большинство современных серверов поставляются с предварительно настроенными каталогами для CGI-программ.

Взаимодействие между браузерами и веб-серверами регулируется протоколом передачи гипертекста (HTTP), который в настоящее время является официальным стандартом Интернета. HTTP использует простую модель запроса и ответа. Например, клиент устанавливает TCP-соединение с сервером и отправляет запрос, сервер отправляет ответ, и соединение закрывается. Эти запросы и ответы имеют форму сообщений, написанных в виде серии простых строк текста.

Сообщения HTTP-запроса и ответа состоят из двух частей. Первый — заголовок, содержащий описательную информацию о запросе или ответе. Различные типы заголовков и их возможное содержимое полностью определяются протоколом HTTP. За заголовком следует пустая строка, затем тело сообщения. Тело сообщения — это фактическое содержимое сообщения, например HTML-страница или GIF-изображение.

## 1. О интерфейсе общего шлюза (CGI)

Common Gateway Interface или сокращенно CGI — это стандарт для подключения различных прикладных программ к веб-страницам. CGI похож на компьютерную программу, которая действует как посредник между стандартами HTML, создающими веб-дисплеи, и другими программами, такими как базы данных. Результаты, полученные в процессе поиска, отправляются обратно на веб-страницу для отображения в формате HTML.

Первоначально CGI представлял собой подход, используемый для приложений программирования на стороне сервера. Обычно используются программы CGI на языках C++ и Perl. CGI — это часть веб-сервера, которая может взаимодействовать с другими программами на сервере. С помощью CGI веб-серверы могут вызывать программы, созданные на различных языках программирования (Common). Взаимодействие между пользователями и различными приложениями, например, базами данных, может быть организовано с помощью CGI (Gateway). Эту возможность CGI можно использовать для веб-сервера IIS.

Протокол HTTP позволяет клиентам и серверам понимать друг друга, передавая всю информацию между ними с помощью заголовков, где каждый заголовок представляет собой пару ключ-значение. При отправке формы программа CGI ищет заголовки, содержащие входную информацию, обрабатывает полученные данные, например, запрашивает ключевые слова, указанные через форму, в базе данных и по готовности возвращает ответ на форму. Клиент отправит специальный заголовок, сообщающий клиенту, какой тип информации ему следует ожидать, а затем саму информацию. Сервер может отправлять дополнительные заголовки, но это необязательно. Смотрите изображение ниже.

CGI часто используется как механизм получения информации от пользователей посредством заполнения форм, доступа к базам данных или создания динамических страниц. Хотя в принципе механизм CGI не имеет уязвимостей в безопасности, программы или скрипты, созданные как CGI, могут иметь уязвимости в безопасности или быть непреднамеренными. Потенциальные уязвимости безопасности, которые могут возникнуть при использовании CGI, включают в себя:

- Злоумышленник может установить CGI-скрипт, который может отправлять файл паролей посетителям, запустившим CGI.
- Программы CGI вызываются несколько раз, в результате чего сервер оказывается перегруженным, поскольку ему приходится запускать несколько программ CGI, которые потребляют память и циклы ЦП веб-сервера.

## 2. Установка языка Perl
Поскольку мы будем включать мод CGI BIN, вам сначала потребуется установить язык Perl. Сервер FreeBSD имеет собственную систему упаковки двоичных файлов. Это облегчит вам установку и управление программным обеспечением. С помощью этой системы пакетов вы также можете управлять установкой модулей Perl.

Введите следующую команду для установки Perl5.

```
root@ns5:~ # pkg install perl5.38-5.38.2_1
```

Приведенная выше команда установит Perl5 на ваш сервер FreeBSD, а двоичный путь к Perl будет /usr/local/bin/Perl.

Поскольку в репозиториях пакетов FreeBSD имеется множество версий Perl, необходимо указать версию Perl в файле /etc/make.conf.

```
root@ns5:~ # ee /etc/make.conf

DEFAULT_VERSIONS+=  perl5=5.30
```

## 3. Включить CGI-мод на Lighttpd
Принципы CGI и способы написания CGI-приложений не являются предметом данного обсуждения. Однако понимание конфигурации CGI помогает понять код mod_cgi, который мы реализуем для настройки lighttpd для запуска процессов CGI.

Прежде чем продолжить обсуждение этой статьи, чтобы упростить настройку CGI-мода на Lighttpd, вам следует прочитать нашу предыдущую статью.

Перед включением мода CGI сначала установим несколько PHP-приложений, которые требуются Lighttpd для включения мода CGI. Введите эту команду, чтобы установить зависимости мода CGI.

```
root@ns5:~ # pkg install php82 mod_php82 php82-mysqli php82-xmlwriter
root@ns5:~ # pkg install php82-gd php82-phar php82-ctype php82-filter php82-iconv php82-curl php82-mysqli php82-pdo php82-tokenizer php82-mbstring php82-session php82-simplexml php82-xml php82-zlib php82-zip php82-dom php82-pdo_mysql php82-ctype
```

Хорошо, теперь перейдем к включению мода CGI. Откройте файл «/usr/local/etc/lighttpd/modules.conf» и поместите в него mod_cgi.

```
server.modules = (
#  "mod_rewrite",
  "mod_access",
#  "mod_auth",
#  "mod_authn_file",
#  "mod_redirect",
#  "mod_setenv",
"mod_alias",
"mod_cgi",
"mod_fastcgi"
)
```

Теперь, когда модуль CGI активен, мы запустим его. Откройте файл «/usr/local/etc/lighttpd/conf.d/cgi.conf» и удалите в нем все скрипты, а затем замените их скриптами, приведенными ниже.

```
$HTTP["url"] =~ "/cgi-bin/" {
cgi.assign                 = ( ".pl"  => "/usr/local/bin/perl",
                               ".cgi" => "/usr/local/bin/perl",
                               ".rb"  => "/usr/bin/ruby",
                               ".erb" => "/usr/bin/eruby",
                               ".py"  => "/usr/local/bin/python" )
}
```

Документы, заканчивающиеся на «.pl», «.cgi» и «.py», будут считаться программами CGI. Переводчики для этой программы будут добавлены в ближайшее время. Следующий шаг — перезапуск Lighttpd.

```
root@ns5:~ # service lighttpd restart
```

## 4. Запуск Lighttpd mod_cgi
После завершения настройки проверьте, может ли указанная выше конфигурация работать на веб-сервере Lighttpd. Вы создаете папку «/usr/local/www/data/cgi-bin», затем создаете файл «arrays.pl».

```
root@ns5:~ # mkdir -p /usr/local/www/data/cgi-bin
root@ns5:~ # cd /usr/local/www/data/cgi-bin
root@ns5:/usr/local/www/data/cgi-bin # touch arrays.pl
root@ns5:/usr/local/www/data/cgi-bin # chown -R www:www /usr/local/www/data/cgi-bin
```

В файл «/usr/local/www/data/cgi-bin/arrays.pl» вставляем следующий скрипт.

```
#!/usr/bin/perl
use strict;
use warnings;

# initializing arrays
my @places = ('Bangalore', 'Chennai', 'Coimbatore');
my @prime_numbers = (2, 3, 5, 7);
my @books;
$books[0] = 'Harry Potter';
$books[1] = 'Sherlock Holmes';
$books[2] = 'To Kill a Mocking Bird';
my @mixed_arr = ('Hello', 1, 'q', @places, $books[0]);

# ----- printing entire array -----
print "\@places: @places\n";
print "\@prime_numbers: @prime_numbers\n";
print "\@mixed_arr: @mixed_arr\n";
# use join to connect array items with preferred characters
print "My favorite books: ".join(', ', @books)."\n\n";

# ----- printing individual item -----
print "First item in \@places: $places[0]\n";
print "Last  item in \@places: $places[-1]\n";
print "Third last item in \@mixed_arr: $mixed_arr[-3]\n";

# ----- printing array slice -----
print "2nd and 3rd item in \@prime_numbers: @prime_numbers[1..2]\n\n";

# ----- array size -----
print "Index of last item in \@places: $#places\n";
my $arr_size = @places;
print "Number of items in \@places:    $arr_size\n\n";

# ----- manipulating array -----
# Add an item, same as $places[$#places + 1]
$places[3] = "Salem";
print "Manually add item: @places\n";

# Delete first item and shift left
shift(@places);
print "shift:             @places\n";
# Delete last item
pop(@places);
print "pop:               @places\n";

# Add an item at end of array
push(@places, "Mysore");
print "push:              @places\n";
# Add an item at beginning of array
unshift(@places, "Hyderabad");
print "unshift:           @places\n";

# change array size by assigning value to $#places
# Change no. of items to 2
$#places = 1;
print "\$#places = 1       @places\n";
# Change no. of items to 0
$#places = -1;
print "\$#places = -1      @places\n";
```

Откройте веб-браузер Google Chrome или Yandex, введите «http://192.168.5.2/cgi-bin/arrays.pl».

Продолжим тестирование файла .cgi. По-прежнему в папке «/usr/local/www/data/cgi-bin» создайте файл «cgiperl.cgi».

```
root@ns5:/usr/local/www/data/cgi-bin # touch cgiperl.cgi
root@ns5:/usr/local/www/data/cgi-bin # chown -R www:www cgiperl.cgi
```

Введите следующий скрипт в файл «/usr/local/www/data/cgi-bin/cgiperl.cgi».

```
#!/usr/local/bin/perl

print "Content-type: text/html\n\n";
print <<htmlcode;
<html>
<head>
<title>CGI Perl Example</title>
</head>
<body>
<h1>CGI Perl Example</h1>
<h3><p>Congratulations on successfully configuring the Perl mod on Apache24!</p><h3>
</body>
```

Снова открываем Google Chrome, вводим «http://192.168.5.2/cgi-bin/cgiperl.cgi». Пожалуйста, посмотрите результаты.

Настройка CGI-скриптов на веб-сервере Lighttpd — относительно простой процесс, который можно выполнить, следуя руководству, содержащемуся в этой статье. Включение поддержки CGI, создание CGI-скриптов, настройка Lighttpd для распознавания скриптов и тестирование скриптов являются важными этапами настройки CGI-скриптов в Lighttpd. С помощью этих шагов вы сможете создавать динамичные и интерактивные веб-страницы, которые можно использовать для предоставления информации или услуг посетителям вашего сайта.

