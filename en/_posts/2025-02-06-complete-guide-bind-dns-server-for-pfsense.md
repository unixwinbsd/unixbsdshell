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
