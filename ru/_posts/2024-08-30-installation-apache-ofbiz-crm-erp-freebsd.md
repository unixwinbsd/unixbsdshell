---
title: "Установка и настройка FreeBSD Apache OFBiz Crm и Erp Platform"
date: 2024-08-30 19:00:00+03:00
excerpt: "Наличие ERP-систем не является ошибочным восприятием, и людей пугает то, что они потеряют работу и будут заменены компьютерами."
id: installation-apache-ofbiz-crm-erp-freebsd
lang: ru
layout: single
categories:
  - FreeBSD
tags: "apache, freebsd, ofbiz, openbsd, configuration, setup, openssl, http, https, ssl"
keywords: apache, freebsd, ofbiz, openbsd, configuration, setup, openssl, http, https, ssl
---

Наличие интегрированной информационной системы является важным и неотложным условием для создания более эффективного и результативного способа работы, который влияет на эффективность усилий по управлению персоналом и улучшение бизнес-процессов. Таким образом, функция управленческого уровня может быть более гибкой в ​​управлении имеющейся информацией при анализе и определении бизнес-стратегий для будущего развития бизнеса компании. По иронии судьбы, внедрение интегрированной информационной системы, такой как ERP (планирование ресурсов предприятия), несет в себе риск неудачи. Информационная система — это компонент, состоящий из людей, информационных технологий и рабочих процедур, которые обрабатывают, хранят, анализируют и распространяют информацию для достижения цели.

Одним из факторов успеха в деловой конкуренции является поддержка использования информационных технологий в управлении бизнес-транзакциями. По этой причине необходимо эффективно управлять информационными технологиями, например, для обеспечения уровня удовлетворенности клиентов и поставщиков.

Оптимальное использование новых информационных технологий (Новых технологий) в управлении ежедневными бизнес-транзакциями для получения информации, которую можно извлечь и использовать в качестве инструмента для принятия стратегических решений. Благодаря интегрированной информационной системе крупномасштабные числовые вычисления могут выполняться с высокой скоростью, обеспечивая относительно быструю связь, хранение больших объемов информации и упрощая к ней доступ, повышая эффективность и результативность работы, представляя информацию информативно, точно, актуально и в соответствии с потребностями руководства и пользователей, автоматизируя полуавтоматические бизнес-процессы и задачи, выполняемые вручную.

Наличие ERP-системы не является ошибочным восприятием, и людей пугает то, что они потеряют работу и будут заменены компьютерами. Напротив, наличие ERP-системы может сделать работу более эффективной и производительной, предоставляя идеи и инновации для принятия стратегических бизнес-решений. для развития компании.

ERP — это аббревиатура, состоящая из трех слов: «enterprise», что означает компанию или организацию, «resource», что означает ресурсы, и «planning», что означает планирование. Таким образом, ERP отражает концепцию, которая приводит к глаголу «планирование», что означает, что ERP (планирование ресурсов предприятия) делает акцент на аспекте планирования. Исходя из вышесказанного, ERP (планирование ресурсов предприятия) — это, в частности, наличие интегрированного аспекта планирования в организации или компании с целью планирования и управления организационными ресурсами, а также возможности адекватного реагирования на потребности клиентов.

Можно сказать, что система ERP — это термин, обозначающий интегрированную информационную систему для поддержки транзакций или операционных процессов в компании.

Перед началом установки Apache OFBiz мы рекомендуем обновить системные порты или пакеты PKG до последних версий. Для обновления системы вы можете выполнить следующую команду.

```
root@ns3:~ # pkg update -f
root@ns3:~ # pkg upgrade -f
```

## 1. Процесс установки Java openjdk17
Перед запуском Apache OFBiz необходимо отметить, что для работы OFBiz требуется Java Development Kit (JDK) openjdk17 или выше. По умолчанию FreeBSD предоставляет репозиторий Java, который можно установить через порты или PKG. Установите Java openjdk17 для запуска OFBiz с помощью следующей команды.

```
root@ns3:~ # pkg install openjdk17
```

## 2. Настройка базы данных OFBiz
Перед установкой Apache OFBiz необходимо подготовить сервер базы данных. Несмотря на то, что в OFBiz встроена база данных Hypersonic SQL, вам не нужно выполнять какую-либо настройку для запуска OFBiz. Однако, если вы хотите использовать базу данных MySQL или MariaDB, или другие коммерческие базы данных, мы рекомендуем сначала установить и создать базу данных для OFBiz. Следуйте инструкциям ниже, чтобы создать базу данных OFBiz с сервером MySQL.

```
root@ns3:~ # mysql -u root -p
Enter password:
root@localhost [(none)]> CREATE DATABASE ofbiz CHARACTER SET utf8;
root@localhost [(none)]> CREATE USER 'userofbiz'@'localhost' IDENTIFIED BY 'router123';
root@localhost [(none)]> GRANT ALL PRIVILEGES ON ofbiz.* TO 'userofbiz'@'localhost';
root@localhost [(none)]> FLUSH PRIVILEGES;
root@localhost [(none)]> exit;

root@localhost [(none)]> CREATE DATABASE ofbizolap CHARACTER SET utf8;
root@localhost [(none)]> CREATE USER 'userofbizolap'@'localhost' IDENTIFIED BY 'router123';
root@localhost [(none)]> GRANT ALL PRIVILEGES ON ofbizolap.* TO 'userofbizolap'@'localhost';
root@localhost [(none)]> FLUSH PRIVILEGES;
root@localhost [(none)]> exit;

root@localhost [(none)]> CREATE DATABASE ofbiztenant CHARACTER SET utf8;
root@localhost [(none)]> CREATE USER 'userofbiztenant'@'localhost' IDENTIFIED BY 'router123';
root@localhost [(none)]> GRANT ALL PRIVILEGES ON ofbiztenant.* TO 'userofbiztenant'@'localhost';
root@localhost [(none)]> FLUSH PRIVILEGES;
root@localhost [(none)]> exit;
```

Вышеуказанный метод создает базу данных ofbiz с именем пользователя и паролем:
database name: ofbiz
database user: userofbiz
database password: router123
database host: 127.0.0.1

database name: ofbizolap
database user: userofbizolap
database password: router123
database host: 127.0.0.1

database name: ofbiztenant
database user: userofbiztenant
database password: router123
database host: 127.0.0.1

## 3. Процесс установки Apache OFBiz
На FreeBSD репозиторий Apache OFBiz недоступен. Вы можете клонировать его на сайте Github или загрузить непосредственно с официального сайта OFBIZ. В этом примере мы клонируем OFBiz напрямую из Github. Выполните следующую команду.

```
root@ns3:~ # cd /usr/local/www
root@ns3:/usr/local/www # git clone https://github.com/apache/ofbiz-framework.git
```

После этого вы переносите OFBiz из Derby в базу данных MySQL, помещая драйвер MySQL JDBC в файл build.gradle. В файле build.gradle добавьте скрипт, указанный ниже.

```
root@ns3:/usr/local/www # cd ofbiz-framework
root@ns3:/usr/local/www/ofbiz-framework # ee build.gradle
dependencies {
         pluginLibsCompile 'mysql:mysql-connector-java:5.1.6'
    }
```

Откройте и отредактируйте файл entityengine.xml, обновите скрипт в файле, используя скрипт, указанный ниже.

```
root@ns3:/usr/local/www/ofbiz-framework # cd framework/entity/config
root@ns3:/usr/local/www/ofbiz-framework/framework/entity/config # ee entityengine.xml
<datasource name="localmysql" helper-class="org.apache.ofbiz.entity.datasource.GenericHelperDAO" field-type-name="mysql" check-on-start="true" add-missing-on-start="true" check-pks-on-start="false" use-foreign-keys="true" join-style="ansi-no-parenthesis" alias-view-columns="false" drop-fk-use-foreign-key-keyword="true" table-type="InnoDB" character-set="utf8" collate="utf8_general_ci">
    <read-data reader-name="tenant" />
    <read-data reader-name="seed" />
    <read-data reader-name="seed-initial" />
    <read-data reader-name="demo" />
    <read-data reader-name="ext" />
    <read-data reader-name="ext-test" />
    <read-data reader-name="ext-demo" />
    <inline-jdbc jdbc-driver="com.mysql.jdbc.Driver" jdbc-uri="jdbc:mysql://localhost/ofbiz?autoReconnect=true&amp;characterEncoding=UTF-8" jdbc-username="ofbiz" jdbc-password="ofbiz" isolation-level="ReadCommitted" pool-minsize="2" pool-maxsize="250" time-between-eviction-runs-millis="600000" />
</datasource>
<datasource name="localmysqlolap" helper-class="org.apache.ofbiz.entity.datasource.GenericHelperDAO" field-type-name="mysql" check-on-start="true" add-missing-on-start="true" check-pks-on-start="false" use-foreign-keys="true" join-style="ansi-no-parenthesis" alias-view-columns="false" drop-fk-use-foreign-key-keyword="true" table-type="InnoDB" character-set="utf8" collate="utf8_general_ci">
    <read-data reader-name="seed" />
    <read-data reader-name="seed-initial" />
    <read-data reader-name="demo" />
    <read-data reader-name="ext" />
    <inline-jdbc jdbc-driver="com.mysql.jdbc.Driver" jdbc-uri="jdbc:mysql://localhost/ofbizolap?autoReconnect=true&amp;characterEncoding=UTF-8" jdbc-username="ofbizolap" jdbc-password="ofbizolap" isolation-level="ReadCommitted" pool-minsize="2" pool-maxsize="250" time-between-eviction-runs-millis="600000" />
</datasource>
<datasource name="localmysqltenant" helper-class="org.apache.ofbiz.entity.datasource.GenericHelperDAO" field-type-name="mysql" check-on-start="true" add-missing-on-start="true" check-pks-on-start="false" use-foreign-keys="true" join-style="ansi-no-parenthesis" alias-view-columns="false" drop-fk-use-foreign-key-keyword="true" table-type="InnoDB" character-set="utf8" collate="utf8_general_ci">
    <read-data reader-name="seed" />
    <read-data reader-name="seed-initial" />
    <read-data reader-name="demo" />
    <read-data reader-name="ext" />
    <inline-jdbc jdbc-driver="com.mysql.jdbc.Driver" jdbc-uri="jdbc:mysql://localhost/ofbiztenant?autoReconnect=true&amp;characterEncoding=UTF-8" jdbc-username="ofbiztenant" jdbc-password="ofbiztenant" isolation-level="ReadCommitted" pool-minsize="2" pool-maxsize="250" time-between-eviction-runs-millis="600000" />
</datasource>
```

Не забудьте заменить derby на базу данных mysql по умолчанию, найдите скрипт делегирования и замените его скриптом ниже.

```
root@ns3:/usr/local/www/ofbiz-framework # cd framework/entity/config
root@ns3:/usr/local/www/ofbiz-framework/framework/entity/config # ee entityengine.xml
<delegator name="default" entity-model-reader="main" entity-group-reader="main" entity-eca-reader="main" distributed-cache-clear-enabled="false">
    <group-map group-name="org.apache.ofbiz" datasource-name="localmysql"/>
    <group-map group-name="org.apache.ofbiz.olap" datasource-name="localmysqlolap"/>
    <group-map group-name="org.apache.ofbiz.tenant" datasource-name="localmysqltenant"/>
</delegator>
     
<delegator name="default-no-eca" entity-model-reader="main" entity-group-reader="main" entity-eca-reader="main" entity-eca-enabled="false" distributed-cache-clear-enabled="false">
    <group-map group-name="org.apache.ofbiz" datasource-name="localmysql"/>
    <group-map group-name="org.apache.ofbiz.olap" datasource-name="localmysqlolap"/>
    <group-map group-name="org.apache.ofbiz.tenant" datasource-name="localmysqltenant"/>
</delegator>
 
<delegator name="test" entity-model-reader="main" entity-group-reader="main" entity-eca-reader="main">
    <group-map group-name="org.apache.ofbiz" datasource-name="localmysql"/>
    <group-map group-name="org.apache.ofbiz.olap" datasource-name="localmysqlolap"/>
    <group-map group-name="org.apache.ofbiz.tenant" datasource-name="localmysqltenant"/>
</delegator>
```

Затем загрузите оболочку Gradle с помощью предоставленного скрипта оболочки. Это загрузит файл gradle-wrapper.jar и поместит его в каталог gradle/wrapper.

```
root@ns3:/usr/local/www/ofbiz-framework # gradle init-gradle-wrapper.sh
```

После этого скомпилируйте ofbiz с помощью следующей команды.

```
root@ns3:/usr/local/www/ofbiz-framework # ./gradlew cleanAll loadAll
```

Последний шаг — запуск службы Apache OFBiz.

```
root@ns3:/usr/local/www/ofbiz-framework # ./gradlew ofbiz
```

Счастливый! Вы успешно установили платформу Apache OFBiz ERP/CRM на сервер FreeBSD 13.2. Наслаждайтесь простотой управления CRM, инвентаризацией, ERP и другими функциями с помощью Apache OFBiz. Продолжайте изучать платформы ERP/CRM в официальном репозитории Apache OFBiz и не стесняйтесь задавать вопросы людям, которые пишут учебные пособия по Apache OFBiz, в разделе комментариев.
