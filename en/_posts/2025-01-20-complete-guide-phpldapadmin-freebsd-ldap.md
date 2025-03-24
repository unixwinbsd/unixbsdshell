---
title: Complete Guide  phpldapadmin PHP Application To Administrator LDAP Over The Web
date: "2025-01-20 15:01:10 +0100"
id: complete-guide-phpldapadmin-freebsd-ldap
lang: en
categories:
  - FreeBSD
tags: "WebServer"
excerpt: phpLDAPadmin is popular among LDAP administrators because it is platform independent. As a web-based application, it allows administrators to maintain LDAP databases remotely from almost any computer
keywords: phpldapadmin, phpmyadmin, freebsd, ldap, php, web, web server, https, ssl
layout: single
author_profile: true
---

phpLDAPadmin is an open source web-based LDAP (Lightweight Directory Access Protocol) administration tool written in PHP. Key features include template-based entry creation; ability to add, modify, rename and delete LDAP entries, user password management with hash support, LDIF (LDAP Data Interchange Format) import/export and browser LDAP schema.

phpLDAPadmin is popular among LDAP administrators because it is platform independent. As a web-based application, it allows administrators to maintain LDAP databases remotely from almost any computer with a web browser. phpLDAPadmin was created by David Smith. Deon George currently maintains the phpLDAPadmin project with the help of various contributors.

If you are going to use phpLDAPadmin from an unsecured public network, start by ensuring that your Apache HTTP server is properly configured to accept HTTPS connections with SSL. If your Apache HTTP server does not have SSL support, your LDAP login and password will be transmitted over the network "in the clear" and could be exposed to unauthorized persons.

In this article we will learn how to install and configure phpLDAPadmin on a FreeBSD 13.2 system.<br><br/>
## 1. PHP and Apache24 Mod Installation
mod_php is an Apache module that allows PHP code to be executed directly by the Apache web server. This means that when a client (such as a web browser) requests a PHP page, the server can execute the PHP code and return the resulting HTML to the client, without requiring a separate PHP language interpreter. This can improve the performance and scalability of PHP applications, as it reduces the costs of starting and managing separate PHP processes. However, this also means that all PHP code runs with the same permissions as the Apache24 web server, which can pose security issues.

In order for phpLDAPadmin to run on a web server, Apache is required with PHP mod. Below is the script to install Apache with PHP mod.

```
root@ns1:~ # pkg install apache24
root@ns1:~ # pkg install php82 mod_php82 php82-mysqli
root@ns1:~ # pkg install php82-gd php82-phar php82-ctype php82-filter php82-iconv php82-curl php82-mysqli php82-pdo php82-tokenizer php82-mbstring php82-session php82-simplexml php82-xml php82-zlib php82-zip php82-dom php82-pdo_mysql php82-ctype
```

In this article we will not discuss Apache24 installation. You can read my previous article which explains how to install Apache24 on FreeBSD. Let's assume that Apache24 is installed properly on FreeBSD. So let's continue by configuring the PHP mod.

The next step is to edit the /usr/local/etc/apache24/httpd.conf file and enter the PHP mod script, as in the following example.

```
<IfModule dir_module>
    DirectoryIndex index.php index.html
</IfModule>

LoadModule php_module         libexec/apache24/libphp.so

<FilesMatch "\.php$">
    SetHandler application/x-httpd-php
</FilesMatch>
<FilesMatch "\.phps$">
    SetHandler application/x-httpd-php-source
</FilesMatch>
AddType application/x-compress .Z
AddType application/x-gzip .gz .tgz

AddType application/x-httpd-php .php
AddType application/x-httpd-php .php .phtml .php3
AddType application/x-httpd-php-source .phps
```

You can put the script at the end/bottom of the /usr/local/etc/apache24/httpd.conf file. Still in the /usr/local/etc/apache24/httpd.conf file, change the following script.

```
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>

Ubah Menjadi

<IfModule dir_module>
    DirectoryIndex index.php index.html
</IfModule>
```

You also add the following script to the bottom of the /usr/local/etc/apache24/httpd.conf file.

```
LoadModule php_module         libexec/apache24/libphp.so

<FilesMatch "\.php$">
    SetHandler application/x-httpd-php
</FilesMatch>
<FilesMatch "\.phps$">
    SetHandler application/x-httpd-php-source
</FilesMatch>
```

After that, we create a php.ini file by copying php.ini-production to the php.ini file, here's how to copy it.

```
root@ns1:~ # cp /usr/local/etc/php.ini-production /usr/local/etc/php.ini
```

After you have the /usr/local/etc/php.ini file, we add the following script.

```
extension=mysqli.so
```

