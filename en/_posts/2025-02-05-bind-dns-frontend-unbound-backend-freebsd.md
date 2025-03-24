---
title: Configuring ISC Bind DNS As Frontend and Unbound Backend For Caching and Forwarding
date: "2025-02-05 15:01:10 +0100"
id: bind-dns-frontend-unbound-backend-freebsd
lang: en
layout: single
author_profile: true
categories:
  - FreeBSD
tags: "DNSServer"
excerpt: An important part of managing server configuration and infrastructure involves maintaining and finding a way to find network interfaces and IP addresses by name website
keywords: freebsd, dns, server, isc, unbound, caching, resolver, backend, frontend
---

An important part of managing server configuration and infrastructure involves maintaining and finding a way to find network interfaces and IP addresses by name website. One way to do this is to set up a Domain Name System (DNS). Use a fully qualified domain name (FQDN). Determining the domain name on a server will make configuration easier, make maintenance easier and improve application service performance.

Setting up private DNS for your private network is a great way to improve your server management and prevent hacker attacks.<br><br/>
## System Specifications
> OS: FreeBSD 13.2-STABLE
> 
> CPU: AMD Phenom II X4 965 3400 MHz
> 
> IP LAN: 192.168.5.2/24
> 
> Domain: unixexplore.com
> 
> IP Unbound: 192.168.5.2
> 
> Port Unbound: 853
> 
> Unbound Version: 1.17.1
> 
> IP DNS Bind: 192.168.5.2
> 
> Port DNS Bind: 53
> 
> DNS Bind Version: BIND 9.18.14 

 In this tutorial, we will set up a bind and unbound DNS server as your domain and private IP address. To carry out the configuration, we will use public DNS from the cloud and Google for the main forward, while we will use unbound DNS as the frontend of the bind DNS server.

 In addition, it is a good idea to also read and practice the article entitled [UNBOUND CONFIGURATION FOR DNSSEC AND DOT CACHING WITH FREEBSD](https://penaadventure.com/en/freebsd/2025/01/11/unbound-caching-dnssec-freebsd-dot/). Because this article is a continuation of the article you are reading. In the article you see the discussion in point "b" namely "Unbound Server as DNS Caching & DNS Over TLS".

In this case, I will not explain how to set unbound, you can read the previous article entitled [UNBOUND IMPLEMENTATION AS DNS OVER TLS CLIENT & SERVER IN FREEBSD](https://www.inchimediatama.org/2024/11/freebsd-unbound-dns-over-tls-dot.html).
<br><br/>
## B. Bind DNS Server Installation and Configuration
The first thing we will discuss is making the DNS Bind server as caching of DNS. On FreeBSD, you can use the pkg command to install it.

```
root@router2:~ # pkg install bind918 bind9-devel bind-tools libuv
```

Enter the following script in the rc.conf file.

```
root@router2:~ # ee /etc/rc.conf
named_enable="YES"
named_program="/usr/local/sbin/named"
named_conf="/usr/local/etc/namedb/named.conf"
#named_chrootdir="/usr/local/etc/namedb"
named_flags="-u -c"
named_uid="bind"
```

Enter the following script in the resolv.conf file.

```
root@router2:~ # ee /etc/resolv.conf
domain unixexplore.com
nameserver 192.168.5.2
```

Create a bind log file.

```
root@router2:~ # mkdir /var/named & mkdir /var/named/log
root@router2:~ # touch /var/named/log/default
root@router2:~ # touch /var/named/log/auth_servers
root@router2:~ # touch /var/named/log/dnssec
root@router2:~ # touch /var/named/log/zone_transfers
root@router2:~ # touch /var/named/log/ddns
root@router2:~ # touch /var/named/log/client_security
root@router2:~ # touch /var/named/log/rate_limiting
root@router2:~ # touch /var/named/log/rpz
root@router2:~ # touch /var/named/log/dnstap
root@router2:~ # touch /var/named/log/queries
root@router2:~ # touch /var/named/log/query-errors
```

Give the Bind program permissions.

```
root@router2:~ # chown -R bind:bind /var/named/log
root@router2:~ # chown -R bind:bind /usr/local/etc/namedb/named.conf
root@router2:~ # chown -R bind:bind /usr/local/etc/namedb/*
```






