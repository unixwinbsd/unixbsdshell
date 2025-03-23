---
title: Aria2 Metalink Guide On FreeBSD To Download Torrent Files
date: "2025-01-14 09:10:00 +0300"
id: install-aria2-metalink-freebsd-torrent
lang: en
layout: single
author_profile: true
categories:
  - FreeBSD
tags: "Anonymous"
excerpt: Aria2 is a utility for downloading torrent files. Supported protocols are HTTP(S), FTP, SFTP, BitTorrent, and Metalink
keywords: torrent, aria, freebsd, metalink, aria2, download, file
---

Aria2 is a utility for downloading torrent files. Supported protocols are HTTP(S), FTP, SFTP, BitTorrent, and Metalink. The advantage of Aria2 is that it can download files from various protocols by utilizing the maximum download bandwidth of your internet network. Another advantage of Aria2 is that it supports downloading files from various protocols such as HTTP(S)/FTP/SFTP and BitTorrent simultaneously, while data downloaded from HTTP(S)/FTP/SFTP is uploaded to the BitTorrent network. By using Metalink chunk checksums, aria2 automatically validates data chunks when downloading files such as BitTorrent.

This article will explain how to configure Aria2 as a tool for downloading torrent files. In practice, we will run Aria2 via the HTTP/HTTPS protocol on ports 80 and 443. With an elegant Web GUI, Aria offers speed and stability in downloading torrent files.


![aria2 rpc host and port](/img/aria2-rpc-host-and-port.jpg)


## 1. Aria2 installation
At the aria2 installation stage, the system used is FreeBSD 13.2 with Lan Server IP 192.168.9.3. Because we will run Aria2 on the HTTP/HTTPS protocol, we need a Web Server such as Apache or NGINX which bridges Aria2 with a web browser so that Aria2 can run on ports 80 and 443.
Before we start installing aria2, we first install apache24, php mod and scgi mod. In this article, we will not explain how to configure Apache24, you can read the previous article which discusses installing and configuring Apache24. OK, let's just start the installation.

```
root@router2:~ # cd /usr/ports/sysutils/screen
root@router2:/usr/ports/sysutils/screen # make install clean

root@router2:~ # cd /usr/ports/lang/php82
root@router2:/usr/ports/lang/php82 # make install clean

root@router2:~ # cd /usr/ports/www/mod_php82
root@router2:/usr/ports/www/mod_php82 # make install clean

root@router2:~ # cd /usr/ports/www/mod_scgi
root@router2:/usr/ports/www/mod_scgi # make install clean

root@router2:~ # cd /usr/ports/security/ca_root_nss
root@router2:/usr/ports/security/ca_root_nss # make install clean
```
After we have successfully installed all the applications above, we continue by installing aria2.

```
root@router2:~ # cd /usr/ports/www/aria2
root@router2:/usr/ports/www/aria2 # make install clean
```

Wait until the installation process is complete. After the installation process is complete, try opening the file /usr/local/etc/rc.d/aria2. If you look at the script it says user and group script. At the end of the installation, aria2 does not create users and groups, unlike apache24 which automatically creates users and groups www. We have to create users and groups for Aria2 manually, the script is as follows.

```
root@router2:~ # pw useradd aria2 -s /sbin/nologin
```

## 2. Configure php and scgi mods
This php and scgi mod is needed so that aria2 can connect to a web browser such as Yandex or Google Chrome. We will configure these two mods with apache24. Now you open the /usr/local/etc/apache24/httpd.conf file, place the following script at the very end or bottom of the httpd.conf file.

```
Alias /webui-aria2/ "/usr/local/www/webui-aria2/docs/"
<Directory "/usr/local/www/webui-aria2/docs">
 AllowOverride None
    Options None
    Require all granted
</Directory>

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

LoadModule scgi_module libexec/apache24/mod_scgi.so
```

After that, create a file /usr/local/etc/apache24/Includes/scgi.conf for the scgi mod (authorization is required for security) and enter the script below.

```
root@router2:~ # touch /usr/local/etc/apache24/Includes/scgi.conf
root@router2:~ # ee /usr/local/etc/apache24/Includes/scgi.conf
<IfModule mod_scgi.c>
SCGIMount /RPC2 192.168.9.3:6800
SCGIServerTimeout 60

 <Location /RPC2>
 Allow from all
 SetHandler scgi-handler
 SCGIServer serv0:6800
 SCGIHandler On
 Options -Multiviews
 </Location>

</IfModule>
```


## 3. Aria2 configuration
The next step is to configure aria2. So that aria2 can run automatically when the computer boots, create the following script in the rc.conf file.

```
root@router2:~ # ee /etc/rc.conf
aria2_enable="YES"
aria2_config="/usr/local/etc/aria2/aria2.conf"
aria2_user="aria2"
aria2_group="aria2"
```

Create a folder to place the downloaded files and the aria2.conf configuration file.

```
root@router2:~ # mkdir -p /usr/local/etc/aria2/downloads
root@router2:~ # mkdir -p /usr/local/etc/aria2/sessions
```

Because at the end of the aria2 installation it does not include a configuration file, namely the aria2.conf file, we create an aria2.conf file in the /usr/local/etc/aria2 folder.

```
root@router2:~ # touch /usr/local/etc/aria2/aria2.conf
root@router2:~ # touch /usr/local/etc/aria2/aria2.log
root@router2:~ # chown -R aria2:aria2 /usr/local/etc/aria2/
```

The chown script above will provide users and groups in the /usr/local/etc/aria2 folder along with the files in that folder. Now we edit the aria2.conf file and enter the following script in the aria2.conf file.


```
root@router2:~ # ee /usr/local/etc/aria2/aria2.conf
dir=/usr/local/etc/aria2/downloads
file-allocation=trunc
continue=true
daemon=true
## Logs
log=/usr/local/etc/aria2/aria2.log
console-log-level=warn
log-level=notice
## Some Limits
max-concurrent-downloads=5
max-connection-per-server=5
min-split-size=20M
split=4
disable-ipv6=true
##
#input-file=/home/user/.aria2/aria2.session
#save-session=/home/user/.aria2/aria2.session
#save-session-interval=30
## RPC
enable-rpc=true
rpc-allow-origin-all=true
rpc-listen-all=true
# Port for JSON RPC
rpc-listen-port=6800
rpc-secret=rVD+dqH5Zcf2tPXH8pWPpz/IctF1OMSt/O+2dCyxMV0=
# Your Certificate file
#rpc-certificate=/usr/local/etc/aria2/aria2.pfx
ca-certificate=/etc/ssl/cert.pem
# Enable encryption
#rpc-secure=true
## Torrents Settings
follow-torrent=mem
follow-metalink=mem
enable-dht6=false
peer-id-prefix=-TR2770-
user-agent=Transmission/2.77
seed-time=0
#seed-ratio=1.0
bt-seed-unverified=true
bt-save-metadata=true
bt-force-encryption=true
# Torrent TCP port
listen-port=6801
# Torrent UDP port
dht-listen-port=6801
```

Next we will create a token that we will use to configure the aria2 Web GUI, because if this token is not created, aria2 will not connect to the Web GUI.

```
root@router2:~ # openssl rand -base64 32
rVD+dqH5Zcf2tPXH8pWPpz/IctF1OMSt/O+2dCyxMV0=
```

To make it easier for you, save the token in Notepad. You enter the "rVD+dqH5Zcf2tPXH8pWPpz/IctF1OMSt/O+2dCyxMV0=" token into the aria2.conf file above in the script rpc-secret=rVD+dqH5Zcf2tPXH8pWPpz/IctF1OMSt/O+2dCyxMV0=


## 4. Aria2 Web Based Control Panel Configuration
This control panel will appear in the Web Browser, with this control panel we can add to delete torrent files and others. To configure the control panel, you can download the file on Github.

```
root@router2:~ # cd /usr/local/www
root@router2:~ # git clone https://github.com/ziahamza/webui-aria2.git
root@router2:~ # chown -R www:www /usr/local/www/webui-aria2
root@router2:~ # chmod -R 777 /usr/local/www/webui-aria2
```

After everything is configured, the next step is to restart/reboot your computer.

```
root@router2:~ # reboot
```

After the computer returns to normal, before we test, you first restart the aria2.

```
root@router2:~ # service aria2 restart
root@router2:~ # service apache24 restart
```

After that, we TEST by downloading the torrent file, open the Yandex Web Browser or Google Chrome, type "http://192.168.9.3/webui-aria2/" . If you don't complete the configuration above, a display like the following image will appear.

![configuration aria2 webui on windows](/img/configuration-aria2-webui-on-windows.jpg)

The image above is the aria2 Web GUI display. We continue by setting up the Web GUI, click the Settings menu then click the Connection Settings menu, you set it as shown in the following image.

![connection setting aria2 rpc host and port](/img/connection-setting-aria2-rpc-host-and-port.jpg)

Enter the host is the FreeBSD Lan Server ip, Enter the port is the rpc port and Enter the secret is the token that we created earlier for the aria2.conf file after that click Save Connection configuration. Now that your aria2 server is RUNNING you can download torrent files as you like. The downloaded file will later be saved in the /usr/local/etc/aria2/downloads folder.
