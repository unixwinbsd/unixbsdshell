---
title: Building MySQL MariaDB on OpenBSD - Installation and Configuration
date: "2024-12-29 11:10:10 +0200"
id: freebsd-mysql-mariadb-bdatabase
lang: en
layout: single
author_profile: true
categories:
  - OpenBSD
tags: "DataBase"
excerpt: Technological advances make things more practical, including database management.
keywords: freebsd, unix, mariadb, mysql, database mysql server, rdmbs
---

If you are someone who works in the database field, of course you are already familiar with MariaDB and MySQL. Both are DBMS or Database Management Systems with almost the same features and functions.

Technological advances make things more practical, including database management. Not just saving time, you will also be given convenience in developing websites or applications. MySQL and MariaDB are the two most popular and frequently used software.

MariaDB is a database management system which is an independent development of MySQL. MariaDB is also called a fork because it is considered another version of MySQL.

Developed in 2009, MariaDB is available as a Relational Database Management Systems application which is still open source. Equipped with many features and maintains compatibility with MySQL. Call it ports and sockets, table definitions, APIs and various protocols.

MariaDB runs on the same Structured Query Language (SQL) as MySQL. Therefore, if you were previously a MySQL user, you will have no difficulty switching to this application.

So, now that you know what MariaDB is, at least you understand a little, right?

As a database management system, MariaDB has several functions, including:
-   Enterprise databases
-   Import Data
-   Big Data Operations
-   Error Message Detection

## 1. Install MariaDB
On the OpenBSD 7.5 operating system, it seems rather difficult to find the MariaDB repository, or even that OpenBSD has replaced it with MaraDB. But it doesn't matter, MaraDB or MySQL come from the same womb. The way to use it is also not much different. To install MariaDB on OpenBSD run the following command.

First, we first look for the Mariadb package which is available in the OpenBSD repository.

```
ns3# pkg_info -Q mariadb
mariadb-client-10.9.8v1
mariadb-server-10.9.8p0v1
mariadb-tests-10.9.8v1
p5-DBD-MariaDB-1.23
```

After that, install the Mariadb package.

```
ns3# pkg_add mariadb-client-10.9.8v1 mariadb-server-10.9.8p0v1
```

### a. Enable Mariadb on OpenBSD
Use the rcctl command to activate Mariadb, you can also use the dos command to enable Mariadb on OpenBSD.

```
ns3# rcctl enable mysqld
ns3# rcctl restart mysqld
mysqld(ok)
ns3# rcctl check mysqld
mysqld(failed)
```

"mysqld(failed)", meaning that Mariadb is not yet running. Try running the command "mysql_install_db".

```
ns3# mysql_install_db
Installing MariaDB/MySQL system tables in '/var/mysql' ...
OK
```

Restart Mariadb.
```
ns3# rcctl restart mysqld
mysqld(ok)
mysqld(ok)
ns3# rcctl check mysqld
mysqld(ok)
```

### b. Create root Password MariaDB
In this section we will create a new root password to secure MariaDB SQL data and remove the insecure default database.

```
ns3# mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] Y
Enabled successfully!
Reloading privilege tables..
 ... Success!


You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n] Y
New password: router123
Re-enter new password: router123
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] Y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] Y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] Y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] Y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

## 2. Test MariaDB Installation
After the installation and configuration process is complete, we carry out testing by connecting the user to the MariaDB database server. 

### a. Check Mariadb version
Before you run other commands, first check the version of Mariadb that you are using in OpenBSD.

```
ns3# mysql -V
mysql  Ver 15.1 Distrib 10.9.8-MariaDB, for OpenBSD (amd64) using readline 4.3
```

### b. Login to MariaDB
Run the command below to log in to the Mariadb server.

