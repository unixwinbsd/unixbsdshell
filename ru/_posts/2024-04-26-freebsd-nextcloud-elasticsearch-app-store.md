---
title: FreeBSD Elasticsearch — как включить полнотекстовый поиск в App Store в NextCloud
date: "2024-04-26 19:00:00 +0300"
id: freebsd-nextcloud-elasticsearch-app-store
lang: ru
layout: single
categories:
  - FreeBSD
excerpt: "Приложения Elasticsearch часто используются организациями или компаниями как средство быстрого и эффективного представления информации."
tags: "nextcloud, freebsd, linux, openbsd, configuration, setup, apt, elasticsearch"
keywords: nextcloud, freebsd, linux, openbsd, configuration, setup, apt, elasticsearch
---

Одной из возможностей Nextcloud является поддержка поиска через веб-интерфейс и в клиенте. В этом режиме поиск осуществляется по именам сравниваемых файлов, а поиск по содержимому файлов не производится. Однако существуют и другие варианты, которые можно использовать для настройки полнотекстового поиска.

Приложения Elasticsearch часто используются организациями или компаниями как средство быстрого и эффективного представления информации. Благодаря своей гибкости и универсальности Elasticsearch стал одной из самых широко используемых и популярных поисковых систем в мире.

На серверах Nextcloud основой полнотекстового поиска является Elasticsearch. Если вы хотите использовать Elasticsearch в качестве поисковой системы, Nextcloud и Elasticsearch необходимо установить отдельно или независимо. Elasticsearch — поисковая система на основе Java, поэтому первым делом необходимо убедиться, что Java установлена. Вам следует проверить, установлена ​​ли Java на вашем сервере FreeBSD.

## Как установить Elasticsearch
Во FreeBSD процесс настройки Elasticsearch очень прост, а процесс установки не слишком сложен. Вам просто нужно установить Elasticsearch через пакет pkg, введя команду ниже.

```
root@ns3:~ # pkg update -f
root@ns3:~ # pkg upgrade -f
root@ns3:~ # pkg install elasticsearch8
```

После этого активируйте elasticsearch7, чтобы он мог запускаться автоматически при перезагрузке компьютера.

```
root@ns3:~ # ee /etc/rc.conf
elasticsearch_enable="YES"
elasticsearch_user="elasticsearch"
elasticsearch_group="elasticsearch"
elasticsearch_config="/usr/local/etc/elasticsearch"
##elasticsearch_config="/usr/local/etc/elasticsearch/elasticsearch.yml"
elasticsearch_login_class="root"
elasticsearch_java_home="/usr/local/openjdk11"
```

Продолжаем установку Java openjdk17.

```
root@ns3:~ # cd /usr/ports/java/openjdk17
root@ns3:~ # make install clean
```

Поскольку мы будем использовать «Elasticsearch» с «полнотекстовым поиском» [Nextcloud,](https://www.inchimediatama.org/2024/11/freebsd-java-openjdk-install.html) для подключения Elasticsearch и полнотекстового поиска потребуется библиотека PHP readline. Выполните следующую команду для установки PHP readline.

```
root@ns3:~ # pkg install php82-readline
```

Следующим шагом будет настройка файла elasticsearch.yml. Включите некоторые скрипты, как в примере ниже.

```
root@ns3:~ # ee /usr/local/etc/elasticsearch/elasticsearch.yml
cluster.name: nextcloud2
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

Добавьте плагин для приема данных Elasticsearch.

```
root@ns3:~ # /usr/local/lib/elasticsearch/bin/elasticsearch-plugin install ingest-attachment
```

Перезапустите Elasticsearch.

```
root@ns3:~ # service elasticsearch restart
```

Проверьте, запущен ли Elasticsearch.

```
root@ns3:~ # curl -XGET '192.168.5.2:9200/?pretty'
```

## Как установить полнотекстовый поиск Nextcloud
После завершения установки Elasticsearch приступайте к установке Nextcloud Full Text Search. Чтобы установить и включить полнотекстовый поиск, следуйте командам ниже.

```
root@ns3:~ # occ app:install files_fulltextsearch
root@ns3:~ # occ app:install fulltextsearch
root@ns3:~ # occ app:install fulltextsearch_elasticsearch
```

После этого активируйте полнотекстовый поиск.

```
root@ns3:~ # occ app:enable files_fulltextsearch
root@ns3:~ # occ app:enable fulltextsearch
root@ns3:~ # occ app:enable fulltextsearch_elasticsearch
```

Теперь вы можете напрямую подключить Nextcloud к Elastic Search. Войдите в свою учетную запись Nextcloud, затем нажмите меню настроек администрирования и меню полнотекстового поиска. Следуйте инструкциям на картинках ниже.

В меню «Адрес сервлета» введите IP-адрес Elasticsearch, то есть «http://192.168.5.2:9200/». В меню индекса также введите «nextcloud2», измените его на скрипт «cluster.name:nextcloud2» в файле elasticsearch.yml.

После этого вы запускаете Генерацию поискового индекса, структура поискового индекса создается в консоли с помощью командной строки, вот пример команды, которую вам нужно выполнить.

```
root@ns3:~ # cd /usr/local/www/nextcloud
root@ns3:/usr/local/www/nextcloud # occ fulltextsearch:index
```

Чтобы убедиться, что полнотекстовый поиск проиндексирован в «App Store» Nextcloud, выполните две командные строки ниже.

```
root@ns3:/usr/local/www/nextcloud # occ fulltextsearch:check
root@ns3:/usr/local/www/nextcloud # occ fulltextsearch:test
```

Последний шаг — запуск веб-сервера Apache.

```
root@ns3:~ # service apache24 restart
```

После настройки этого раздела на сайте Nextcloud ваш сервер Nextcloud теперь подключен к Elasticsearch. Вы можете выполнять поиск эффективно, безопасно и с высокой производительностью. Мало того, Elasticsearch также способен быстро анализировать данные в больших объемах.



