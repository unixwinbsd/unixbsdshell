---
title: Tutorial MariaDB on OpenBSD 7 - Installlation and Configuration
date: "2025-02-11 20:05:11 +0200"
id: tutorilas-mariadb-installation-configuration
lang: en
layout: single
author_profile: true
categories:
  - OpenBSD
tags: "WebServer"
excerpt: MariaDB is a branch of MySQL. The developers developed this RDBMS to maintain the structure and features of MySQL
keywords: mariadb, rdbms, mysql, database, openbsd, unix
---

In website development, databases play a very important role in storing and managing data. Well, the two most popular databases today are MySQL and MariaDB.

Although they have a similar structure, both have different features and elements. Therefore, you should understand the differences between MariaDB and MySQL so that you can choose which one best suits your needs and optimize the performance of your web application.

## 1. What is MariaDB?
MariaDB is a branch of MySQL. The developers developed this RDBMS to maintain the structure and features of MySQL.

Because, at that time there was an acquisition process by Oracle, the company behind Oracle Database which is MySQL's biggest competitor. It is feared that after this acquisition MySQL will not be continued anymore.

MariaDB developers ensure that each release is compatible with the related MySQL version. MariaDB not only adopts MySQL data files and table definitions, but also uses the same client protocol, client API, port, and socket so that MySQL users can switch to MariaDB without any problems. Then, just like MySQL, MariaDB can be modified using SQL statements.

MariaDB is currently one of the popular relational database management systems (RDBMS). The MariaDB database is open source licensed under the GNU GPL 2, and is ready to be used by communities and companies with a long history and extensive knowledge to manage and maintain it.

There are two packages of MariaDB on OpenBSD.
1.  **mariadb-server:**  The MariaDB server.
2.  **mariadb-client:**  The client side of MariaDB server including mysqlclient library and headers for the MariaDB client API.

## 2. How to find MariaDB packages
Before you start installing MariaDB on OpenBSD, you should pay attention to which version you are going to install. To find out all versions of MariaDB that are in the OpenBSD package you can run the pkg_info command.

```
hostname1# pkg_info -Q mariadb
```

Or you can also use the following command.

```
hostname1# pkg_info -Q mariadb | grep php
```

## 3. Install Database MariaDB Server
Once you are sure about the version of MariaDB to be installed, let's start the installation process.

```
Install MariaDB Server
hostname1# pkg_add mariadb-server
quirks-6.122 signed on 2023-09-01T21:25:11Z
mariadb-server-10.9.4v1:(...): ok
mariadb-server-10.9.4v1: ok
The following new rcscripts were installed: /etc/rc.d/mysqld
See rcctl(8) for details.
New and changed readme(s):
	/usr/local/share/doc/pkg-readmes/mariadb-server
```

If you want to install the MariaDB client, run the following command.

```
hostname1# pkg_add -v mariadb-client
```

To enable and control daemons and services on OpenBSD use the rcctl command.

```
hostname1# rcctl enable mysqld
hostname1# rcctl start mysqld
```

After that you run the mysql_install_db command to create the necessary system tables and binary files.

```
hostname1# mysql_install_db
```

## 4. Check if MariaDB port is open
This check is very important, because we can make sure whether the MariaDB service is running or not. The check can be done by checking mysqld or also by checking the MariaDB port.

```
hostname1# rcctl check mysqld
```

Run the grep command to search for the mysql application process.

```
hostname1# pgrep mysqld
```

You can also use the following command to check the MariaDB application running on OpenBSD.

```
hostname1# ps aux | grep mysqld
```

If you want to check the open tcp ports, run the following command.

```
hostname1# netstat -f inet -na
hostname1# netstat -f inet -na | grep 3306
```

## 5. Configure MariaDB
Run the mysql_secure_installation command to remove the database and any insecure default settings that could compromise your production server. Select the desired settings when prompted and set a strong root password.

```
hostname1# mysql_secure_installation
```

Next step, you Enable the mysql socket in /var/run/mysql.sock and allow MariaDB to listen for client connections on port 3306. Open /etc/my.cnf in your favorite editor.

```
hostname1# nano /etc/my/cnf
[client-server]
socket=/var/run/mysql/mysql.sock
port=3306
```

## 6. Login MariaDB
In the section above you have created a username and password that we will use to log in to the MariaDB server. Okay now let's go straight to the MariaDB server.

```
Login to MariaDB
hostname1# mysql -u root -p
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 11
Server version: 10.9.4-MariaDB OpenBSD port: mariadb-server-10.9.4v1

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

Enter the root password you created in the previous section.

After you have successfully logged in to the MariaDB database server, we will try to practice by creating a database.

```
MariaDB [(none)]> CREATE DATABASE openbsd \
CHARACTER SET utf8mb4 \
COLLATE utf8mb4_unicode_ci;
```

create user.

```
MariaDB [(none)]> GRANT ALL PRIVILEGES \
    ON openbsd.* \
    TO mary@'localhost' \
    IDENTIFIED BY 'mary123';
```

reload privileges

```
MariaDB [(none)]> FLUSH PRIVILEGES;
```

Congratulations, you have successfully installed and configured MariaDB on OpenBSD 7, and the database server can connect with other applications installed on the server, including web stacks such as Nginx, httpd, and apache2-httpd via run time sockets and PHP.
