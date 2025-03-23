---
title: CONFIGURATION UNBOUND FOR CACHING DNSSEC AND DOT WITH FREEBSD
date: "2025-01-11 11:16:10 +0400"
id: unbound-caching-dnssec-freebsd-dot
lang: en
layout: single
categories:
  - FreeBSD
tags: "unbound, dns, server, dnssec, freebsd"
excerpt: Unbound is a highly secure validating, recursive, and caching DNS server developed by NLnet Labs, VeriSign Inc, Nominet, and Kirei
keywords: freebsd, unbound, dns, server, security, caching, dot, doh, dnssec, configuration, openbsd
---
  
The DNS service or Domain Name System is a basic service of the Internet, as well as other networks that operate based on the TCP/IP protocol family, and is used to obtain matching host names on the network with corresponding digital addresses. Despite such a simple description, DNS is perhaps the most complex network service in terms of its structure and set of interactions, and its reliable operation depends on the reliable operation of everyone and everything.

DNSSEC provides authentication of DNS records using digital signatures, which protects them from possible replacement. However, at the same time, all data is always transmitted in clear form and is not protected from view during transit in any way. In the absence of a digital signature, if desired, it can be easily modified for various purposes - from criminal to censorship.  

This article explains how to configure and use the Unbound DNS server, both as caching, DNSSEC and as DNS over TLS.  

Unbound is a highly secure validating, recursive, and caching DNS server developed by NLnet Labs, VeriSign Inc, Nominet, and Kirei. This software is distributed free of charge under the BSD license. Binaries are written with a high security focus and very strict C code.  

To help improve online privacy, Unbound supports DNS-over-TLS and DNS-over-HTTPS allowing clients to encrypt their communications. In addition, it supports various modern standards that limit the amount of data exchanged with official servers. This standard not only improves privacy but also helps make DNS more robust. The most important are Query Name Minimisation, Aggressive Use of DNSSEC-Validated Cache and support for authority zones, which can be used to load a copy of the root zone.
<br><br/>

## 1. System Specifications
- OS: FreeBSD 13
- Hostname: miner4
- Domain: miner4pool.org
- IP LAN Private: 192.168.9.3/24
- Unbound Caching DNS Server: 192.168.9.3@53
- Unbound Caching DNS over TLS: 192.168.9.3@853
<br><br/>
## 2. Instalasi Unbound
Generally, to install Unbound on a FreeBSD server, you can do two things, namely ports and pkg. In this article, we will discuss installation with pkg.  

The first step, please log in to your FreeBSD server via the FreeBSD console or remote SSH with PuTTy, then run the following command lines:  

```
root@miner4:~ # pkg update
root@miner4:~ # pkg upgrade
```

After finishing updating the pkg package, we continue with installing unbound, type the following command to install unbound.  

```
root@miner4:~ #  pkg install unbound bind-tools ca_root_nss
```

So that the unbound server can run automatically every time the FreeBSD server is turned off or restarted, you must first add the following command to rc.conf, by typing the script below in the /etc/rc.conf file.  

```
root@miner4:~ # ee /etc/rc.conf
unbound_enable="YES"
unbound_config="/usr/local/etc/unbound/unbound.conf"
unbound_pidfile="/usr/local/etc/unbound/unbound.pid"
unbound_anchorflags=""
```

After that, in the "resolv.conf" file, enter the script below. Use the default FreeBSD application, namely "ee" to add the script.

```
root@miner4:~ # ee /etc/resolv.conf
domain miner4pool.org
nameserver 192.168.9.3
```  

Edit the hosts file, use the command ee /etc/hosts, then enter the following syntax.
  
```
root@miner4:~ # ee /etc/hosts
127.0.0.1      localhost localhost.miner4pool.org
192.168.9.3    miner4 miner4.miner4pool.org
```
 <br><br/>
## 3. Unbound Configuration
Before we start configuring unbound.conf, the unbound application requires a "root.hints" file that lists the primary DNS Server. The Unbound DNS Server application comes with a list of root DNS Servers in its code, but it ensures an up-to-date copy on each server. A good practice is to update this file every six months.

By default the unbound.conf file is in the /usr/local/etc/unbound directory. In the Putty console, type the following command.

```
root@miner4:~ # wget ftp://FTP.INTERNIC.NET/domain/named.cache -O /usr/local/etc/unbound/root.hints
```

Additionally, Unbound requires an auto-trust-anchor file. This file contains the keys so that DNSSEC can be validated. To generate the root.key, run the following command in the putty console.
  
```
root@miner4:~ # cd /usr/local/etc/unbound
root@miner4:~ # unbound-anchor -a "/usr/local/etc/unbound/root.key"
```

The next step is to create the necessary keys for Unbound to be controlled by unbound-control, type the command below to run unbound control setup.

```
root@miner4:~ # cd /usr/local/etc/unbound
root@miner4:~ # unbound-control-setup
```

