---
title: How to Install phpMyAdmin on FreeBSD with Apache24
date: "2025-01-16 11:10:00 +0300"
id: install-phpmyadmin-on-freebsd-apache
lang: en
layout: single
categories:
  - FreeBSD
tags: "WebServer"
excerpt: phpMyAdmin supports multiple database servers. It can also import data from various formats such as CSV and SQL.
keywords: php, phpmyadmin, freebsd, server, openbsd, apache, install
---


This article presents a simple way on how to install phpMyAdmin on FreeBSD 13.2 Stable, phpMyAdmin is a free and open source application written in PHP to manage MySQL and MariaDB tasks such as creating, changing, deleting, dropping database tables, or running other database management commands and supports most of MySQL features.

phpMyAdmin supports multiple database servers. This application can also import data from various formats such as CSV and SQL, and also export data to various other formats such as CSV, SQL, XML, PDF, ISO / IEC 26300 - OpenDocument Text and Spreadsheets, Word, Excel, LATEX and others. This application is free, open source, and written using PHP.

This article will not explain how to install Apache Web Server, about how to install Apache you can read the previous article on this blogspot. In this article we assume that our FreeBSD system has apache24 installed.

To start installing phpMyAdmin in this tutorial, the tip is to use the FreeBSD system ports. Installing phpMyAdmin with system ports shouldn't be too difficult. You can simply follow the steps in this article.


# System Specifications
- OS: FreeBSD13.2 Stable
- IP LAN Server: 192.168.9.3
- Apache: apache24 (In this article we will not discuss it, we will assume it is already installed)
- MySQL: mysql80-server (In this article we will not discuss it, we will assume it is already installed)
php: php82
- phpMyAdmin: phpMyAdmin5-php82


## 1. phpMyAdmin installation
Before we get into the installation process, once again I remind you that this tutorial does not discuss the installation of apache24 and mysql80-server, you can read the previous article that discusses these two applications. Now let's start installing phpMyAdmin, but first you must install php82 first.


```
root@router2:~ # cd /usr/ports/lang/php82
root@router2:/usr/ports/lang/php82 # make install clean
```
After completing the installation of php82, then we continue by installing phpMyAdmin.

```
root@router2:~ # cd /usr/ports/databases/phpmyadmin5
root@router2:/usr/ports/databases/phpmyadmin5 # make install clean
```
The display below is the display when the phpMyAdmin installation is complete.

```
Installing phpMyAdmin5-php82-5.2.1_1...
===> Creating groups.
Using existing group 'www'.
phpMyAdmin5-php82-5.2.1_1 has been installed into:

    /usr/local/www/phpMyAdmin

Please edit config.inc.php to suit your needs.

To make phpMyAdmin available through your web site, I suggest
that you add something like the following to httpd.conf:

Alias /phpmyadmin/ "/usr/local/www/phpMyAdmin/"

<Directory "/usr/local/www/phpMyAdmin/">
        Options None
        AllowOverride Limit

        Require local
        Require host .example.com
</Directory>

SECURITY NOTE: phpMyAdmin is an administrative tool that has had several
remote vulnerabilities discovered in the past, some allowing remote
attackers to execute arbitrary code with the web server's user credential.
All known problems have been fixed, but the FreeBSD Security Team strongly
advises that any instance be protected with an additional protection layer,
e.g. a different access control mechanism implemented by the web server
as shown in the example.  Do consider enabling phpMyAdmin only when it
is in use.
===>  Cleaning for php82-ctype-8.2.6
===>  Cleaning for php82-ctype-8.2.6
===>  Cleaning for php82-filter-8.2.6
===>  Cleaning for php82-filter-8.2.6
===>  Cleaning for php82-iconv-8.2.6
===>  Cleaning for php82-iconv-8.2.6
===>  Cleaning for php82-mysqli-8.2.6
===>  Cleaning for php82-mysqli-8.2.6
===>  Cleaning for php82-session-8.2.6
===>  Cleaning for php82-session-8.2.6
===>  Cleaning for php82-xml-8.2.6
===>  Cleaning for php82-xml-8.2.6
===>  Cleaning for php82-xmlwriter-8.2.6
===>  Cleaning for php82-xmlwriter-8.2.6
===>  Cleaning for php82-bz2-8.2.6
===>  Cleaning for php82-bz2-8.2.6
===>  Cleaning for php82-gd-8.2.6
===>  Cleaning for php82-gd-8.2.6
===>  Cleaning for php82-mbstring-8.2.6
===>  Cleaning for php82-mbstring-8.2.6
===>  Cleaning for oniguruma-6.9.8_1
===>  Cleaning for oniguruma-6.9.8_1
===>  Cleaning for php82-zip-8.2.6
===>  Cleaning for php82-zip-8.2.6
===>  Cleaning for libzip-1.9.2
===>  Cleaning for libzip-1.9.2
===>  Cleaning for php82-zlib-8.2.6
===>  Cleaning for php82-zlib-8.2.6
===>  Cleaning for phpMyAdmin5-php82-5.2.1_1
===>  Cleaning for phpMyAdmin5-php80-5.2.1_1
===>  Cleaning for php81-8.1.19
===>  Cleaning for php81-8.1.19
===>  Cleaning for php81-ctype-8.1.19
===>  Cleaning for php81-ctype-8.1.19
===>  Cleaning for php81-filter-8.1.19
===>  Cleaning for php81-filter-8.1.19
===>  Cleaning for php81-iconv-8.1.19
===>  Cleaning for php81-iconv-8.1.19
===>  Cleaning for php81-mysqli-8.1.19
===>  Cleaning for php81-mysqli-8.1.19
===>  Cleaning for php81-session-8.1.19
===>  Cleaning for php81-session-8.1.19
===>  Cleaning for php81-xml-8.1.19
===>  Cleaning for php81-xml-8.1.19
===>  Cleaning for php81-bz2-8.1.19
===>  Cleaning for php81-bz2-8.1.19
===>  Cleaning for php81-gd-8.1.19
===>  Cleaning for php81-gd-8.1.19
===>  Cleaning for php81-mbstring-8.1.19
===>  Cleaning for php81-mbstring-8.1.19
===>  Cleaning for oniguruma-6.9.8_1
===>  Cleaning for oniguruma-6.9.8_1
===>  Cleaning for php81-zip-8.1.19
===>  Cleaning for php81-zip-8.1.19
===>  Cleaning for libzip-1.9.2
===>  Cleaning for libzip-1.9.2
===>  Cleaning for php81-zlib-8.1.19
===>  Cleaning for php81-zlib-8.1.19
===>  Cleaning for phpMyAdmin5-php81-5.2.1_1
root@router2:/usr/ports/databases/phpmyadmin5 #
```

The script above will appear after the phpMyAdmin installation process is complete, what you should pay attention to is the red writing. Okay, let's continue the configuration process, after you have finished installing phpMyAdmin, the next step is to copy the config.sample.inc.php file into the config.inc.php file. The file is located in the /usr/local/www/phpMyAdmin folder.

```
root@router2:~ # cd /usr/local/www/phpMyAdmin
root@router2:/usr/local/www/phpMyAdmin # cp config.sample.inc.php config.inc.php
```

## 3. Edit httpd.conf File
In this third step we will edit the file /usr/local/etc/apache24/httpd.conf. Editing this file will provide the url to access phpMyAdmin. Insert the red script above into the httpd.conf file. Place the script below at the very bottom or at the very end of the httpd.conf script.

```
Alias /phpmyadmin "/usr/local/www/phpMyAdmin/"
<Directory "/usr/local/www/phpMyAdmin">
    AllowOverride None
    Options None
    Require all granted
</Directory>
```

## 4. Apache24 mod php installation
Now we continue by installing the php mod so that it can run on the apache24 web server.

```
root@router2:~ # cd /usr/ports/www/mod_php82
root@router2:/usr/ports/www/mod_php82 # make install clean

root@router2:~ # cd /usr/ports/databases/php82-mysqli
root@router2:/usr/ports/databases/php82-mysqli # make install clean
```
After the two applications above are installed, we continue by editing the httpd.conf file, in the script below.

```
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>
```
Add the index.php script, so the script looks like the following.

```
<IfModule dir_module>
    DirectoryIndex index.php index.html
</IfModule>
```
Still in the httpd.conf file, place the following script at the end or bottom of the httpd.conf file script.

```
LoadModule php_module         libexec/apache24/libphp.so

<FilesMatch "\.php$">
    SetHandler application/x-httpd-php
</FilesMatch>
<FilesMatch "\.phps$">
    SetHandler application/x-httpd-php-source
</FilesMatch>
```

Next, we create a php.ini file, by copying the php.ini-production file to php.ini. The file is located in the /usr/local/etc folder.

```
root@router2:~ # cp /usr/local/etc/php.ini-production /usr/local/etc/php.ini
```
After that, in the /usr/local/etc/php.ini file, we add the following script.

```
extension=mysqli.so
```
Create a new file named info.php and place it in the /usr/local/www/apache24/data folder. In the info.php file, insert the following script "<?php phpinfo(); ?>"

```
root@router2:~ # touch /usr/local/www/apache24/data/info.php
root@router2:~ # ee /usr/local/www/apache24/data/info.php
<?php phpinfo(); ?>
```

After that, do the test by typing the url: http://192.168.9.3/info.php, in the Google Chrome or Yandex address bar menu.

We also tested phpMyAdmin by typing the url: http://192.168.9.3/phpmyadmin/ in the Yandex or Google Chrome browser.

Use the username and password from MySQL Server to login.
