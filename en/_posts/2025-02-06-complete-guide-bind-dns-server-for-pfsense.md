---
title: Complete Guide to BIND DNS Server Settings for PFSense
date: "2025-02-06 09:17:10 +0300"
id: complete-guide-bind-dns-server-for-pfsense
lang: en
layout: single
author_profile: true
categories:
  - FreeBSD
tags: "DNSServer"
excerpt: The existence of the ISC-Bind plugin can be felt by PFSense users who have slow internet networks.
keywords: pfsense, isc, dns, bind, freebsd, setup, dns server, unbound
---

The PFSense firewall has many functions, its job is not only as a router that provides internet services to clients. With the various features it has, PFSense's functions can be improved. Because many of the default FreeBSD plugins have been modified by PFSense, such as the ISC-Bind DNS server.

The existence of the ISC-Bind plugin can be felt by PFSense users who have slow internet networks. With the help of a DNS server that is capable of caching and serving name servers, your internet speed will increase. Because every client who accesses the internet from your PFSense no longer looks for a DNS server, only the PFSense database serves the DNS request.

At PFSense there are lots of DNS application services that you can use, such as Unbound and DNS Forwarding. However, in this article we dedicate it to explaining in detail the installation and configuration process for the ISC-Bind DNS server. Not only that, this article also discusses name server services that can be utilized by DHCP servers.<br><br/>
## 1. Basic Configuration
Before we go any further, on how to install and configure ISC-Bind. The first step you have to do is set the hostname and PFsense resolver.

![pfsense system information](https://www.opencode.net/unixbsdshell/statif-page/-/raw/main/pfsense_system_information.jpg)

After that, you continue to click the System ->> General Setup menu. Please set it according to your PFSense computer specifications. For more details, look at the image below.

![general setup pfsense](https://www.opencode.net/unixbsdshell/statif-page/-/raw/main/general_setup_pfsense.jpg)

What you need to pay attention to here is "DNS Resolution Behavior", don't make the wrong choice. For other settings, just leave the default.<br><br/>
## 2. Install DNS server ISC-Bind
If you use ISC-Bind as a server serving DNS requests, PFSense provides convenience. Why?, because PFSense has provided the official ISC-Bind repository. The repository is taken from the FreeBSD repository, but has been modified to be used in PFSense. OK, now let's just install ISC-Bind, click  **System ->> Package Manager >>> Available Packages**. Then look for the bind application, continue by pressing the green Install button.

After the installation process is complete, it's time to start configuring. We start by clicking the  **Services ->> Bind DNS Server.** For the settings, you can see the following image.

![bind-setting-on-pfsense](https://www.opencode.net/unixbsdshell/statif-page/-/raw/main/bind_setting_pfsense_1.jpg)

![bind-setting-on-pfsense](https://www.opencode.net/unixbsdshell/statif-page/-/raw/main/bind_setting_pfsense_2.jpg)

![bind-setting-on-pfsense](https://www.opencode.net/unixbsdshell/statif-page/-/raw/main/bind_setting_pfsense_3.jpg)

In the image above, what you need to underline is the settings in the "Forwarder IPs" and "Custom Options" options. To set "Custom Options", you type the script below.

```
allow-query { localnets; };
allow-query-cache { localnets; };
allow-transfer { none; };
empty-zones-enable yes;
auth-nxdomain no;
```

After that we move on to setting "ACLs". Remove all Bind PFSense default "ACLs". Then you create new "ACLs". Follow the instructions we explained in the image to create new "ACLs". In this example, we will create two ACLs:

 - localnets
 - localhosts

![bind acls pfsense](https://www.opencode.net/unixbsdshell/statif-page/-/raw/main/bind_acls_pfsense.jpg)

![edit bind acls pfsense](https://www.opencode.net/unixbsdshell/statif-page/-/raw/main/edit_bind_acls_pfsense.jpg)

The next step is to make settings in "Views". See the image below for a guide to setting up "Views".

![bind general option](https://www.opencode.net/unixbsdshell/statif-page/-/raw/main/bind_general_option.jpg)


## 3. Setup Zones 
The final step is to set the "Zones". We will create 4 Zones, namely kursor.my.id, localhost, 7.168.192.in-addr.arpa and 127.in-addr.arpa.  

### a. Zone kursor.my.id  

> Zone Name: kursor.my.id
> Zone Type: Master
> View: dnspfsense
> Name Server: ns5.kursor.my.id

![pfsense zona domain](https://www.opencode.net/unixbsdshell/statif-page/-/raw/main/pfsense_zona_domain.jpg)

### b. Zone localhost
> Zone Name: localhost
> Zone Type: Master
> View: dnspfsense
> Name Server: ns5.localhost
> allow-transfer: localhosts

![zona domain record pfsense](https://www.opencode.net/unixbsdshell/statif-page/-/raw/main/zona_domain_record_pfsense.jpg)

### c. Zone 7.168.192.in-addr.arpa
> Zone Name: 7.168.192.in-addr.arpa
> Zone Type: Master
> View: dnspfsense
> Name Server: ns5.kursor.my.id

![pfsense domain zone](https://www.opencode.net/unixbsdshell/statif-page/-/raw/main/pfsense_domain_zone.jpg)

### d. Zone 127.in-addr.arpa
> Zone Name: 127.in-addr.arpa
> Zone Type: Master
> View: dnspfsense
> Name Server: ns5.localhost
> allow-transfer: localhosts

![record zna domain](https://www.opencode.net/unixbsdshell/statif-page/-/raw/main/record_zna_domain.jpg)

##  4. Test Zones
This section is also very important, because we will check each zone to see whether the zone can contain a serial number. Before we check the zone, create a symlink first with the command below.

```
[2.7.2-RELEASE][root@ns5.kursor.my.id]/root: ln -s /var/etc/named/etc/namedb /etc
```

We will check the main configuration file first, namely named.conf. Run the following command to check the named.conf file.

```
[2.7.2-RELEASE][root@ns5.kursor.my.id]/root: named-checkconf /etc/namedb/named.conf
```

After that, check each zone. Run the command below to check the zone.

```
[2.7.2-RELEASE][root@ns5.kursor.my.id]/root: named-checkzone kursor.my.id /etc/namedb/master/dnspfsense/kursor.my.id.DB
zone kursor.my.id/IN: loaded serial 2714365806
OK
```

```
[2.7.2-RELEASE][root@ns5.kursor.my.id]/root: named-checkzone 7.168.192.in-addr.arpa /etc/namedb/master/dnspfsense/7.168.192.in-addr.arpa.DB
zone 7.168.192.in-addr.arpa/IN: loaded serial 2714374177
OK
```

```
[2.7.2-RELEASE][root@ns5.kursor.my.id]/root: named-checkzone localhost /etc/namedb/master/dnspfsense/localhost.DB
zone localhost/IN: loaded serial 2714373907
OK
```

```
[2.7.2-RELEASE][root@ns5.kursor.my.id]/root: named-checkzone 127.in-addr.arpa /etc/namedb/master/dnspfsense/127.in-addr.arpa.DB
zone 127.in-addr.arpa/IN: loaded serial 2714374263
OK
```

## 5. Test Bind DNS Server  
After we have checked all the zones and there are no problems, it's time to check the BIND DNS server. We will use the dig and nslookup commands to check BIND. Run the following command to check the BIND DNS Server.

```
[2.7.2-RELEASE][root@ns5.kursor.my.id]/root: dig yahoo.com

; <<>> DiG 9.18.19 <<>> yahoo.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 25968
;; flags: qr rd ra; QUERY: 1, ANSWER: 6, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 6c557890f8ed5b2401000000662f72862924cfbb3f954331 (good)
;; QUESTION SECTION:
;yahoo.com.			IN	A

;; ANSWER SECTION:
yahoo.com.		1800	IN	A	74.6.143.26
yahoo.com.		1800	IN	A	98.137.11.164
yahoo.com.		1800	IN	A	74.6.231.21
yahoo.com.		1800	IN	A	98.137.11.163
yahoo.com.		1800	IN	A	74.6.143.25
yahoo.com.		1800	IN	A	74.6.231.20

;; Query time: 34 msec
;; SERVER: 192.168.7.1#53(192.168.7.1) (UDP)
;; WHEN: Mon Apr 29 10:12:22 UTC 2024
;; MSG SIZE  rcvd: 162
```

```
[2.7.2-RELEASE][root@ns5.kursor.my.id]/root: nslookup google.com
Server: 192.168.7.1 Address: 192.168.7.1#53 Non-authoritative answer:
google.com	canonical name = forcesafesearch.google.com.
Name:	forcesafesearch.google.com
Address: 216.239.38.120
Name:	forcesafesearch.google.com
Address: 2001:4860:4802:32::78
```

## 6. Create Dynamic DNS For DHCP Server
After making sure that your BIND DNS server is running normally on the PFSense machine, we continue by creating "Dynamic DNS". To do this, click  Services ->> DHCP Server. To make it easier to explain, please see the picture below.

![lan general dhcp pfsense](https://www.opencode.net/unixbsdshell/statif-page/-/raw/main/lan_general_dhcp_pfsense.jpg)

![server option pfsense](https://www.opencode.net/unixbsdshell/statif-page/-/raw/main/server_option_pfsense.jpg)

![dynamic dns pfsense](https://www.opencode.net/unixbsdshell/statif-page/-/raw/main/dynamic_dns_pfsense.jpg)

## 7. Script Generated by BIND DNS Server PFSense
So that this article is complete and serves as learning material for you, below we will attach an example of the BIND DNS server script.
> /var/etc/named/etc/namedb/named.conf
```
#Bind pfsense configuration
#Do not edit this file!!!

 key "rndc-key" {
 	algorithm hmac-sha256;
 	secret "4vgWNftJMHE59MIczVG+I0OkMTAE0GHGi1TNQmgbv/4=";
 };

 controls {
 	inet 127.0.0.1 port 8953
 		allow { 127.0.0.1; } keys { "rndc-key"; };
 };



options {
	directory "/etc/namedb";
	pid-file "/var/run/named/pid";
	statistics-file "/var/log/named.stats";
	max-cache-size 256M;
	dnssec-validation yes;

	listen-on-v6 port 53 { none; };
	listen-on port 53 { 192.168.7.1;  };
	forwarders { 1.1.1.1; 8.8.8.8; };
	allow-query { localnets; };
	allow-query-cache { localnets; };
	allow-transfer { none; };
	empty-zones-enable yes;
	auth-nxdomain no;
};

logging { category default { null; }; };

acl "localhosts" {
	127.0.0.1;
};

view "dnspfsense" { 
	recursion yes;
	match-clients { localnets; localhosts; };
	allow-recursion { localnets; localhosts; };

	zone "kursor.my.id" {
		type master;
		file "/etc/namedb/master/dnspfsense/kursor.my.id.DB";
		allow-query { none; };
		allow-transfer { none; };
		allow-update { none; };
	};

	zone "localhost" {
		type master;
		file "/etc/namedb/master/dnspfsense/localhost.DB";
		allow-query { none; };
		allow-transfer { localhosts; };
		allow-update { none; };
	};

	zone "7.168.192.in-addr.arpa" {
		type master;
		file "/etc/namedb/master/dnspfsense/7.168.192.in-addr.arpa.DB";
		allow-query { none; };
		allow-transfer { none; };
		allow-update { none; };
	};

	zone "127.in-addr.arpa" {
		type master;
		file "/etc/namedb/master/dnspfsense/127.in-addr.arpa.DB";
		allow-query { none; };
		allow-transfer { localhosts; };
		allow-update { none; };
	};

};
```

> /var/etc/named/etc/namedb/master/dnspfsense/kursor.my.id.DB
```
$TTL 43200
;
$ORIGIN kursor.my.id.

;	Database file kursor.my.id.DB for kursor.my.id zone.
;	Do not edit this file!!!
;	Zone version 2714365806
;
kursor.my.id.	 IN  SOA ns5.kursor.my.id. 	 zonemaster.kursor.my.id. (
		2714365806 ; serial
		1d ; refresh
		2h ; retry
		4w ; expire
		1h ; default_ttl
		)

;
; Zone Records
;
@ 	 IN NS 	ns5.kursor.my.id.
ns5 	 IN A  	192.168.7.1
```

> /var/etc/named/etc/namedb/master/dnspfsense/7.168.192.in-addr.arpa.DB
```
$TTL 43200
;
$ORIGIN 7.168.192.in-addr.arpa.

;	Database file 7.168.192.in-addr.arpa.DB for 7.168.192.in-addr.arpa zone.
;	Do not edit this file!!!
;	Zone version 2714374177
;
7.168.192.in-addr.arpa.	 IN  SOA ns5.kursor.my.id. 	 zonemaster.7.168.192.in-addr.arpa. (
		2714374177 ; serial
		1d ; refresh
		2h ; retry
		4w ; expire
		1h ; default_ttl
		)

;
; Zone Records
;
@ 	 IN NS 	ns5.kursor.my.id.
100 	 IN PTR  	ns5.kursor.my.id.
```

> /var/etc/named/etc/namedb/master/dnspfsense/localhost.DB
```
$TTL 43200
;
$ORIGIN localhost.

;	Database file localhost.DB for localhost zone.
;	Do not edit this file!!!
;	Zone version 2714373907
;
localhost.	 IN  SOA ns5.localhost. 	 zonemaster.localhost. (
		2714373907 ; serial
		1d ; refresh
		2h ; retry
		4w ; expire
		1h ; default_ttl
		)

;
; Zone Records
;
@ 	 IN NS 	ns5.localhost.
ns5 	 IN A  	127.0.0.1
```

> /var/etc/named/etc/namedb/master/dnspfsense/127.in-addr.arpa.DB
```
$TTL 43200
;
$ORIGIN 127.in-addr.arpa.

;	Database file 127.in-addr.arpa.DB for 127.in-addr.arpa zone.
;	Do not edit this file!!!
;	Zone version 2714374263
;
127.in-addr.arpa.	 IN  SOA ns5.localhost. 	 zonemaster.127.in-addr.arpa. (
		2714374263 ; serial
		1d ; refresh
		2h ; retry
		4w ; expire
		1h ; default_ttl
		)

;
; Zone Records
;
@ 	 IN NS 	ns5.localhost.
1.0.0 	 IN PTR  	localhost.
```

You currently have a BIND DNS server on the PFSense machine. You can experience internet speed with BIND DNS server. Maybe, you feel satisfied now, because your internet is not slow.
