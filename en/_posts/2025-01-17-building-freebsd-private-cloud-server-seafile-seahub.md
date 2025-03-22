---
title: Building a FreeBSD Private Cloud Server With Seafile Seahub Server
date: "2025-01-17 12:10:00 +0300"
id: building-freebsd-private-cloud-server-seafile-seahub
lang: en
layout: single
categories:
  - FreeBSD
tags: "SysAdmin"
excerpt: Seafile is a high performance and best file sharing application solution. Seafile open source cloud storage is written using the Python Django framework
keywords: freebsd, seafile, seahub, building, cloud, server, private, setup, blog, http
---

Seafile is a cross-platform self-hosted cloud storage and sync solution. In other words, Seafile is very similar to Dropbox or Google Drive except you have full control over your platform instance. As such, Seafile operates in direct competition with Nextcloud and Owncloud.

The Seafile software system comes in three parts: server software, desktop and mobile sync applications, and Drive software. The latter is desktop only and creates a virtual drive to access and upload files stored on your Seafile server.

Seafile is a high performance and best file sharing application solution. Seafile open source cloud storage is written using the Python Django framework and functions very similarly to Dropbox and Google Drive clouds. Seafile open source file storage server consists of WYSIWYG Markdown editing, Wiki, file labels, from django Seahub, Seafile server and Ccnet server.

One of the advantages of Seafile is that its features are no less than Dropbox or Google Drive. One of Seafile's features includes file encryption, version control, two-factor authentication, online editing, file locking, backup, data recovery, and much more. When looking at the Github site, the hosted open source file sharing software Seafile has 8.7k GitHub stars and 1.3k GitHub forks. It's really amazing data.

On FreeBSD systems Seafile is often used as a self-hosted file synchronization and sharing solution and as a cross-platform cloud file hosting service that can be used to store files on a central server.

Seafile file collections are synchronized with personal computers or mobile device clients separately. Files or libraries can also be encrypted with a user-selected password. Seafile file sharing tool also allows users to create groups and easily share files into groups. Seafile manages files using libraries and each library has an owner. Owners can share libraries with other users or with groups with read-only or read-write permissions.

Its fantastic features have attracted the attention of Open source fans to make Seafile an alternative to Dropbox or Google Drive. With various advantages and a multitude of users, it would not be wrong if in this article we will discuss the installation and configuration process of Seafile as a private cloud.

# System Specifications
1. OS FreeBSD 13.3
2. IP Private FreeBSD: 192.168.5.2
3. Host dan Domain: ns1@unixexplore.com 
4. Apache24 sebagai Reverse PRoxy
5. Seahub version: Seahub-9.0.10_1
6. Seafile version: Seafile-9.0.10

## 1. Fusefs Kernel Installation

Before we install Seafile, the fusefs kernel must be installed first, because if the fusefs kernel is not installed the Seafile server will not run perfectly. To activate the fusefs kernel, type the following command in the /boot/loader.conf file.

```
root@ns1:~ # ee /boot/loader.conf
fusefs_load="YES"
fuse_load="YES"
```

Then so that the kernel can be loaded automatically, type the following script in the /etc/rc.conf file. Use the "ee" editor or the "nano" editor to enter the following script.

```
root@ns1:~ # ee /etc/rc.conf
kld_list="fuse"
kld_list="fusefs"
kld_list="/boot/kernel/fusefs.ko"
```

## 2. Seafile and Seahub installation

To start installing Seafile, you should use the ports system in FreeBSD. Update the ports first before installing Seafile.

```
root@ns1:~ # portupgrade -af
root@ns1:~ # portmaster -af
```

After you have finished updating the ports, type the following command to start the Seafile installation.

```
root@ns1:~ # cd /usr/ports/net-mgmt/seafile-server
root@ns1:/usr/ports/net-mgmt/seafile-server # make install clean
====> Compressing man pages (compress-man)
===> Staging rc.d startup script(s)
===>  Installing for seafile-server-9.0.10
===>  Checking if seafile-server is already installed
===>   Registering installation for seafile-server-9.0.10
Installing seafile-server-9.0.10...
===> Creating groups.
Using existing group 'seafile'.
===> Creating users
Using existing user 'seafile'.
net-mgmt/seafile-server is a dependency of Seafile. To install the
full application, www/seahub must be installed.
```

Once complete, continue with the Seahub installation.

```
root@ns1:/usr/ports/net-mgmt/seafile-server # cd /usr/ports/www/seahub
root@ns1:/usr/ports/www/seahub # make install clean
===>   Registering installation for seahub-9.0.10
Installing seahub-9.0.10...
===> Creating groups.
Using existing group 'seafile'.
===> Creating users
Using existing user 'seafile'.
To generate a new config,
cd /usr/local/www/haiwen/seafile-server
Then choose sqlite or mysql, mysql needs to be setup with root.
sqlite: ./setup-seafile.sh
mysql: ./setup-seafile-mysql.sh
Further instructions will be provided after the setup script is finished.


If a new config is created, run this to setup admin.

/usr/local/www/haiwen/seafile-server/reset-admin.sh


Don't forget to run update scripts after every minor and major update located in
	/usr/local/www/haiwen/seafile-server/upgrade

Patch updates (5.1.x) don't require these scripts, however do required a restarted.
```

When installing Seafile and Seahub, the fusefs-libs library is usually not installed completely. Type the following command to install the fusefs-libs library.

```
root@ns1:~ # find /usr/local/lib -name '*.la' | xargs grep -l 'libiconv\.la' | xargs pkg which
```

Then update portmaster to see the version of fusefs-lib used.

```
root@ns1:~ # portmaster -L | grep fusefs-lib
===>>> fusefs-libs-2.9.9_2
```

Try looking at the version of fusefs-lib, from the example script above the version of fusefs-lib is fusefs-libs-2.9.9_2. After the fusefs-libs version type the following command.

```
root@ns1:~ # portmaster -o sysutils/fusefs-libs fusefs-libs-2.9.9_2
```

## 3. Seafile and Seahub configuration
The first step that must be done to configure Seafile is to activate the SQL database, type the following command to activate SQLite3.

```
root@ns1:~ # cd /usr/local/www/haiwen/seafile-server
root@ns1:/usr/local/www/haiwen/seafile-server # ./setup-seafile.sh
-----------------------------------------------------------------
This script will guide you to config and setup your seafile server.

Make sure you have read seafile server manual at 

	https://download.seafile.com/published/seafile-manual/home.md

Note: This script will guide your to setup seafile server using sqlite3,
which may have problems if your disk is on a NFS/CIFS/USB.
In these cases, we suggest you setup seafile server using MySQL.

Press [ENTER] to continue
-----------------------------------------------------------------

Checking packages needed by seafile ...

Checking python on this machine ...
Find python: python3.9

  Checking python module: python-sqlite3 ... Done.

Checking for sqlite3 ...Done.

Checking Done.

What would you like to use as the name of this seafile server?
Your seafile users will be able to see the name in their seafile client.
You can use a-z, A-Z, 0-9, _ and -, and the length should be 3 ~ 15
[server name]: unixexplore

What is the ip or domain of this server?
For example, www.mycompany.com, or, 192.168.1.101

[This server's ip or domain]: 192.168.5.2

What tcp port do you want to use for seafile fileserver?
8082 is the recommended port.
[default: 8082 ] 8082

This is your config information:

server name:        unixexplore
server ip/domain:   192.168.5.2
seafile data dir:   /usr/local/www/haiwen/seafile-data
fileserver port:    8082

If you are OK with the configuration, press [ENTER] to continue.

Generating ccnet configuration in /usr/local/www/haiwen/ccnet...

Generating seafile configuration in /usr/local/www/haiwen/seafile-data ...

-----------------------------------------------------------------
Seahub is the web interface for seafile server.
Now let's setup seahub configuration. Press [ENTER] to continue
-----------------------------------------------------------------

Creating database now, it may take one minute, please wait... 

/usr/local/www/haiwen/seafile-server

Done.

creating seafile-server-latest symbolic link ... done

-----------------------------------------------------------------
Your seafile server configuration has been completed successfully.
-----------------------------------------------------------------

run seafile server:     sysrc seafile_enable=YES
                        service seafile { start | stop | restart }
run seahub  server:     sysrc seahub_enable=YES
                        service seahub { start | stop | restart }
run reset-admin:        ./reset-admin.sh

-----------------------------------------------------------------
If the server is behind a firewall, remember to open these tcp ports:
-----------------------------------------------------------------

port of seafile fileserver:   8082
port of seahub:               8000

When problems occur, refer to

      https://download.seafile.com/published/seafile-manual/home.md

for more information.
```

After that restart Seafile and Seahub.

```
root@ns1:~ # service seafile restart
Stopping seafile.
Starting seafile server, please wait ...
** Message: 21:29:16.716: seafile-controller.c(678): No seafevents.

Seafile server started

root@ns1:~ # service seahub restart
Seahub is not running
LC_ALL is not set in ENV, set to en_US.UTF-8
Starting seahub at port 8000 ...

Seahub is started
```

Create a Seafile user and password.

```
root@ns1:~ # cd /usr/local/www/haiwen/seafile-server
root@ns1:/usr/local/www/haiwen/seafile-server # ./reset-admin.sh
E-mail address: datainchi@gmail.com
Password: ketikkan passwor
Password (again): ketikkan password (harus sama dengan atas)
Superuser created successfully.
```

Next, type the following command.

```
root@ns1:/usr/local/www/haiwen/seafile-server # pgrep -f seafile-controller
3217
root@ns1:/usr/local/www/haiwen/seafile-server # pgrep -f seahub
3233
3235
3236
3237
3238
3239
```

After all the installation is complete, now you can set up a private cloud server from the Seafile server. You can directly use the server to synchronize files, add users and groups, and share files between them or with the public without relying on external services.
