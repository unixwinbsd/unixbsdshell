---
title: FreeBSD OpenSearch — настройка подключаемых модулей OpenSearch TLS и сертификатов с помощью OpenSSL
date: "2022-09-02 09:00:00 +0300"
id: configuration-freebsd-opensearch-certificates-openssl
lang: ru
layout: single
categories:
  - FreeBSD
excerpt: "Opensearch можно объединить с платформами доставки данных Beat (Filebeat, Winlogbeat и т. д.), благодаря чему пользователи OpenSearch могут выстроить полный цикл управления журналами в виде сбора, систематизации и поиска."
tags: "freebsd, opensearch, app, configuration, openssl, certificate"
keywords: freebsd, opensearch, app, configuration, openssl, certificate
---

OpenSearch — это утилита с открытым исходным кодом, которая часто используется в качестве поисковой системы, а также как мощный инструмент для анализа данных. Opensearch написан на основе кодовой базы Elasticsearch 7.10.2. Системы Opensearch включают в себя:
- Хранилище и поисковая система.
- Веб-интерфейс.
- Среда визуализации данных OpenSearch Dashboard.
- А также дополнения, которые позволяют использовать ряд функций поисковой системы Elasticsearch.

OpenSearch появился благодаря участию сообщества и поддержке крупных компаний, таких как Red Hat, SAP, Capital One и т. д. Система Opensearch часто используется для полнотекстового поиска на сайтах, поскольку она может хранить и анализировать логи, а также визуализировать и анализировать полученную информацию.

Opensearch можно объединить с платформами доставки данных Beat (Filebeat, Winlogbeat и т. д.), благодаря чему пользователи OpenSearch могут выстроить полный цикл управления журналами в виде сбора, систематизации и поиска.

# Технические характеристики используемой системы
- OS: FreeBSD 13.3
- Hostname: ns3
- IP address: 192.168.5.2
- Logstash version: logstash8-8.11.3
- Opensearch version: Opensearch-2.11.1
- Dependencies: bash jna

Java version:
- openjdk version "17.0.9" 2023-10-17
- OpenJDK Runtime Environment (build 17.0.9+9-1)
- OpenJDK 64-Bit Server VM (build 17.0.9+9-1, mixed mode, sharing)

## А. Настройка транспортного уровня TLS
В этом разделе мы объясним, как настроить плагин безопасности Opensearch во FreeBSD. В Opensearch есть много плагинов, которые вы можете использовать, все эти плагины требуют SSL-сертификат, который может поддерживать множество дополнительных функций и методов настройки. Вы можете создать конфигурацию сертификата TLS, используя такие инструменты, как OpenSSL.

```
root@ns3:~ # cd /usr/local/etc/opensearch
root@ns3:/usr/local/etc/opensearch # rm -f *pem
```

### а. Сгенерировать корневой сертификат
Прежде чем двигаться дальше, давайте сначала создадим закрытый ключ для корневого сертификата. Корневой сертификат используется для подписи других сертификатов.

```
root@ns3:/usr/local/etc/opensearch # openssl genrsa -out root-ca-key.pem 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
.........................................+++++
.............................+++++
e is 65537 (0x010001)
```

После этого вы создаете самоподписанный корневой сертификат CA. Используйте параметр -subj, чтобы предоставить информацию, специфичную для вашего хоста.

```
root@ns3:/usr/local/etc/opensearch # openssl req -new -x509 -sha256 -key root-ca-key.pem -out root-ca.pem -days 730
```

### б. Создать сертификат администратора
Далее создайте сертификат для администратора. Чтобы создать сертификат администратора, сначала создайте новый ключ, чтобы администратор мог выполнять административные задачи, связанные с плагином безопасности.

```
root@ns3:/usr/local/etc/opensearch # openssl genrsa -out admin-key-temp.pem 2048
```

Чтобы сертификат администратора можно было использовать с программами Java, преобразуйте ключ в формат PKCS.

```
root@ns3:/usr/local/etc/opensearch # openssl pkcs8 -inform PEM -outform PEM -in admin-key-temp.pem -topk8 -nocrypt -v1 PBE-SHA1-3DES -out admin-key.pem
```

Далее создайте запрос на подпись сертификата (CSR) для сертификата администратора (Certificate Signing Request) на основе закрытого ключа. Этот файл действует как заявка в центр сертификации на подписанный вами сертификат.

```
root@ns3:/usr/local/etc/opensearch # openssl req -new -key admin-key.pem -out admin.csr
```

Теперь, когда закрытый ключ и запрос на подпись созданы, подпишите сертификат администратора, используя корневой сертификат и закрытый ключ, созданные ранее.

```
root@ns3:/usr/local/etc/opensearch # openssl x509 -req -in admin.csr -CA root-ca.pem -CAkey root-ca-key.pem -CAcreateserial -sha256 -out admin.pem -days 730
```

### C. Создать сертификат узла
Создание узла сертификата практически идентично действиям по созданию сертификата администратора, описанным выше. В этом разделе мы сгенерируем ключи и CSR с новыми именами файлов для каждого узла и клиентского сертификата. Чтобы создать сертификат узла или клиента, сначала создайте новый ключ.

Добавление сертификата TLS обеспечит дополнительную безопасность вашего хоста Opensearch. Сертификат TLS предоставит клиенту разрешение на проверку подлинности узла и шифрование трафика между клиентом и хостом. Сертификат TLS настраивается в файле opensearch.yml, и мы сохраним его в каталоге /usr/local/etc/opensearch. Чтобы создать самоподписанный сертификат, выполните следующие действия.

```
root@ns3:/usr/local/etc/opensearch # openssl genrsa -out node1-key-temp.pem 2048
```

Затем преобразуйте закрытый ключ в формат PKCS, чтобы его могли использовать приложения Java.

```
root@ns3:/usr/local/etc/opensearch # openssl pkcs8 -inform PEM -outform PEM -in node1-key-temp.pem -topk8 -nocrypt -v1 PBE-SHA1-3DES -out node1-key.pem
```

Затем создайте новый CSR для сертификата хоста на основе закрытого ключа.

```
root@ns3:/usr/local/etc/opensearch # openssl req -new -key node1-key.pem -out node1.csr
```
Для всех сертификатов хоста и клиента необходимо указать альтернативное имя субъекта (SAN). Перед созданием подписанного сертификата создайте файл расширения SAN, содержащий имя хоста и доменное имя.

```
root@ns3:/usr/local/etc/opensearch # echo 'subjectAltName=DNS:node1.datainchi.com' > node1.ext
```

Далее создайте сертификат.

```
root@ns3:/usr/local/etc/opensearch # openssl x509 -req -in node1.csr -CA root-ca.pem -CAkey root-ca-key.pem -CAcreateserial -sha256 -out node1.pem -days 730 -extfile node1.ext
```
После этого вы конвертируете корневой сертификат CA в формат .crt, чтобы корневой сертификат CA можно было установить на сервере Opensearch.

```
root@ns3:/usr/local/etc/opensearch # openssl x509 -outform der -in root-ca.pem -out root-ca.crt
```

Измените владельца и разрешения каталога на opensearch:opensearch.

```
root@ns3:/usr/local/etc/opensearch # chown -R opensearch:opensearch /usr/local/etc/opensearch/
```

### г. Подготовка сертификатов для Opensearch
После создания всех сертификатов приступайте к их установке и добавлению в файл конфигурации OpenSearch, а именно «opensearch.yml». Откройте «/usr/local/etc/opensearch/opensearch.yml» и в самом низу скрипта (в конце скрипта) добавьте приведенный ниже скрипт.

```
root@ns3:/usr/local/etc/opensearch # ee opensearch.yml
plugins.security.ssl.transport.pemcert_filepath: /usr/local/etc/opensearch/node1.pem
plugins.security.ssl.transport.pemkey_filepath: /usr/local/etc/opensearch/node1-key.pem
plugins.security.ssl.transport.pemtrustedcas_filepath: /usr/local/etc/opensearch/root-ca.pem
plugins.security.ssl.http.enabled: true
plugins.security.ssl.http.pemcert_filepath: /usr/local/etc/opensearch/node1.pem
plugins.security.ssl.http.pemkey_filepath: /usr/local/etc/opensearch/node1-key.pem
plugins.security.ssl.http.pemtrustedcas_filepath: /usr/local/etc/opensearch/root-ca.pem
plugins.security.allow_default_init_securityindex: true
plugins.security.authcz.admin_dn:
  - 'CN=A,OU=UNIT,O=ORG,L=TORONTO,ST=ONTARIO,C=CA'
plugins.security.nodes_dn:
  - 'CN=node1.datainchi.com,OU=UNIT,O=ORG,L=TORONTO,ST=ONTARIO,C=CA'
plugins.security.audit.type: internal_opensearch
plugins.security.enable_snapshot_restore_privilege: true
plugins.security.check_snapshot_restore_write_privileges: true
```

## B. Процесс установки плагинов
Как мы знаем, Opensearch включает в себя несколько основных плагинов, входящих в состав его установочного пакета. В Opensearch плагины используются для улучшения основных функций и возможностей. Помимо основных плагинов, вы также можете писать свои собственные плагины, а на Github также доступны плагины, созданные сообществом.

Для того чтобы функции плагина работали безупречно и подключались к Opensearch, все плагины должны иметь доступ к данным в кластере. Прежде чем использовать плагин, вы должны сначала понять его функции и преимущества. Итак, какие плагины доступны в Opensearch? Вы можете просмотреть доступные плагины, выполнив команду ниже.

```
root@ns3:~ # cd /usr/local/lib/opensearch/bin
root@ns3:/usr/local/lib/opensearch/bin # ./opensearch-plugin list
analysis-extension
analysis-fess
analysis-icu
configsync
minhash
opensearch-alerting
opensearch-anomaly-detection
opensearch-asynchronous-search
opensearch-cross-cluster-replication
opensearch-custom-codecs
opensearch-geospatial
opensearch-index-management
opensearch-job-scheduler
opensearch-knn
opensearch-ml
opensearch-neural-search
opensearch-notifications
opensearch-notifications-core
opensearch-observability
opensearch-performance-analyzer
opensearch-reports-scheduler
opensearch-security
opensearch-security-analytics
opensearch-sql
```

Полный список дополнительных поддерживаемых плагинов:

- repository-gcs
Добавлена ​​поддержка сервиса Google Cloud Storage в качестве хранилища снимков.

- analysis-icu
Добавлен модуль Lucene ICU с расширенной поддержкой Unicode и использованием библиотек ICU. Этот модуль обеспечивает улучшенный анализ азиатских языков, нормализацию Unicode, преобразование регистра Unicode, поддержку сопоставления и транслитерацию.

- repository-azure
Добавлена ​​поддержка хранилища BLOB-объектов Azure в качестве репозитория моментальных снимков.

- analysis-kuromoji
Добавлен модуль анализа куромодзи Lucene для японского языка.

- analysis-nori
Добавлен модуль анализа Lucene nori для Кореи.

- analysis-phonetic
Предоставляет фильтр токенов, который преобразует выражения в их фонетическое представление с помощью алгоритмов Soundex, Metaphone и других.

- repository-s3
Добавлена ​​поддержка AWS S3 в качестве репозитория снимков.

- analysis-smartcn
Добавлен модуль анализа Smart Chinese Lucene для китайского текста или смешанного китайско-английского текста.

- analysis-stempel
Добавлен модуль анализа штампов Lucene для польского языка.

- ingest-attachment
Извлекайте вложения файлов в распространенных форматах (таких как PPT, XLS и PDF) с помощью библиотеки извлечения текста Apache Tika.

- mapper-annotated-text
Индексирует текст, представляющий собой комбинацию обычного текста и специальной разметки. Эта комбинация используется для идентификации объектов, таких как люди или организации.

- mapper-murmur3
Вычисляет хэш значения поля на основе времени индекса и сохраняет его в индексе.

- mapper-size
Предоставляет размер поля метаданных, который индексирует размер источника в байтах.

- repository-hdfs
Добавлена ​​поддержка файловой системы HDFS в качестве репозитория снимков.

- transport-nio
Неблокируемая клиент-серверная сетевая библиотека, созданная с использованием Netty.

## C. Как использовать плагин Opensearch
Функциональность и возможности Opensearch можно расширить, добавив пользовательские плагины. Примеры использования плагинов, которые могут улучшить функциональность Opensearch, например, плагины, которые могут добавлять пользовательские типы сопоставлений, скрипты движка и т. д. В этом разделе мы узнаем, как использовать плагин Opensearch. Вот пример скрипта для установки плагина.

/usr/local/lib/opensearch/bin/opensearch-plugin установить имя-плагина.

Чтобы прояснить ваше понимание плагинов, здесь мы покажем пример того, как установить плагин.

```
root@ns3:~ # cd /usr/local/lib/opensearch/bin
root@ns3:/usr/local/lib/opensearch/bin # ./opensearch-plugin install org.codelibs.opensearch:opensearch-analysis-fess:2.9.0
root@ns3:/usr/local/lib/opensearch/bin # ./opensearch-plugin install org.codelibs.opensearch:opensearch-analysis-extension:2.9.0
root@ns3:/usr/local/lib/opensearch/bin # ./opensearch-plugin install org.codelibs.opensearch:opensearch-minhash:2.9.0
root@ns3:/usr/local/lib/opensearch/bin # ./opensearch-plugin install org.codelibs.opensearch:opensearch-configsync:2.9.0
root@ns3:/usr/local/lib/opensearch/bin # ./opensearch-plugin install analysis-icu
root@ns3:/usr/local/lib/opensearch/bin # ./opensearch-plugin install org.opensearch.plugin:opensearch-anomaly-detection:2.2.0.0
```

## D. Как удалить плагины
Как только вы научитесь устанавливать плагины, вам наверняка захочется узнать, как их удалять. Скрипт почти такой же, как и при установке плагина, вот синтаксис для удаления плагина.

/usr/local/lib/opensearch/bin/opensearch-plugin удалить имя-плагина.

Ниже мы покажем вам, как удалить плагины.

```
root@ns3:/usr/local/lib/opensearch/bin # ./opensearch-plugin remove org.codelibs.opensearch:opensearch-configsync:2.9.0
root@ns3:/usr/local/lib/opensearch/bin # ./opensearch-plugin remove analysis-icu
```

В этой статье мы подробно объяснили процесс создания SSL-сертификата, а также как установить и удалить плагин Opensearch. Мы рекомендуем вам прочитать все страницы документации Opensearch, чтобы расширить свои знания и упростить использование функций и возможностей Opensearch.
