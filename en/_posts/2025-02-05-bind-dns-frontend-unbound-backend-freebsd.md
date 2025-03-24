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

 



