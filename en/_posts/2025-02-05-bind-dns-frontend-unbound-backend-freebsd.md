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

Next, we edit the named.conf file. But before we edit the named.conf file, we have to make sure in the unbound.conf file that port 53 is off. Because in the previous tutorial with the title "CONFIGURATION UNBOUND DNS SERVER CACHING DNSSEC AND DNS OVER TLS WITH FREEBSD" port 53 in the unbound.conf file was open. Now we close port 53 in the unbound.conf file. Why does port 53 have to be turned off in the unbound.conf file, because we will use port 53 in the bind application to listen to the port.

Example of a complete script for the /usr/local/etc/unbound/unbound.conf file with port 53 in an inactive or dead state.

```
root@router2:~ # ee /usr/local/etc/unbound/unbound.conf

#
# Example configuration file.
#
# See unbound.conf(5) man page, version 1.17.1.
#
# this is a comment.

# Use this anywhere in the file to include other text into this file.
#include: "otherfile.conf"

# Use this anywhere in the file to include other text, that explicitly starts a
# clause, into this file. Text after this directive needs to start a clause.
#include-toplevel: "otherfile.conf"

# The server clause sets the main parameters.
server:
	# whitespace is not necessary, but looks cleaner.

	# verbosity number, 0 is least verbose. 1 is default.
	verbosity: 1

	# print statistics to the log (for every thread) every N seconds.
	# Set to "" or 0 to disable. Default is disabled.
	# statistics-interval: 0

	# enable shm for stats, default no.  if you enable also enable
	# statistics-interval, every time it also writes stats to the
	# shared memory segment keyed with shm-key.
	# shm-enable: no

	# shm for stats uses this key, and key+1 for the shared mem segment.
	# shm-key: 11777

	# enable cumulative statistics, without clearing them after printing.
	# statistics-cumulative: no

	# enable extended statistics (query types, answer codes, status)
	# printed from unbound-control. Default off, because of speed.
	# extended-statistics: no

	# Inhibits selected extended statistics (qtype, qclass, qopcode, rcode,
	# rpz-actions) from printing if their value is 0.
	# Default on.
	# statistics-inhibit-zero: yes

	# number of threads to create. 1 disables threading.
	num-threads: 2

	# specify the interfaces to answer queries from by ip-address.
	# The default is to listen to localhost (127.0.0.1 and ::1).
	# specify 0.0.0.0 and ::0 to bind to all available interfaces.
	# specify every interface[@port] on a new 'interface:' labelled line.
	# The listen interfaces are not changed on reload, only on restart.
	#interface: 192.168.5.2
	#interface: 127.0.0.1
	interface: 192.168.5.2@853
	interface: 127.0.0.1@853

	# interface: 192.0.2.154@5003
	# interface: 2001:DB8::5
	# interface: eth0@5003

	# enable this feature to copy the source address of queries to reply.
	# Socket options are not supported on all platforms. experimental.
	# interface-automatic: no

	# instead of the default port, open additional ports separated by
	# spaces when interface-automatic is enabled, by listing them here.
	# interface-automatic-ports: ""

	# port to answer queries from
	#port: 53

	# specify the interfaces to send outgoing queries to authoritative
	# server from by ip-address. If none, the default (all) interface
	# is used. Specify every interface on a 'outgoing-interface:' line.
	# outgoing-interface: 192.0.2.153
	# outgoing-interface: 2001:DB8::5
	# outgoing-interface: 2001:DB8::6

	# Specify a netblock to use remainder 64 bits as random bits for
	# upstream queries.  Uses freebind option (Linux).
	# outgoing-interface: 2001:DB8::/64
	# Also (Linux:) ip -6 addr add 2001:db8::/64 dev lo
	# And: ip -6 route add local 2001:db8::/64 dev lo
	# And set prefer-ip6: yes to use the ip6 randomness from a netblock.
	# Set this to yes to prefer ipv6 upstream servers over ipv4.
	# prefer-ip6: no

	# Prefer ipv4 upstream servers, even if ipv6 is available.
	# prefer-ip4: no

	# number of ports to allocate per thread, determines the size of the
	# port range that can be open simultaneously.  About double the
	# num-queries-per-thread, or, use as many as the OS will allow you.
	# outgoing-range: 4096

	# permit Unbound to use this port number or port range for
	# making outgoing queries, using an outgoing interface.
	# outgoing-port-permit: 32768

	# deny Unbound the use this of port number or port range for
	# making outgoing queries, using an outgoing interface.
	# Use this to make sure Unbound does not grab a UDP port that some
	# other server on this computer needs. The default is to avoid
	# IANA-assigned port numbers.
	# If multiple outgoing-port-permit and outgoing-port-avoid options
	# are present, they are processed in order.
	# outgoing-port-avoid: "3200-3208"

	# number of outgoing simultaneous tcp buffers to hold per thread.
	# outgoing-num-tcp: 10

	# number of incoming simultaneous tcp buffers to hold per thread.
	# incoming-num-tcp: 10

	# buffer size for UDP port 53 incoming (SO_RCVBUF socket option).
	# 0 is system default.  Use 4m to catch query spikes for busy servers.
	so-rcvbuf: 1m

	# buffer size for UDP port 53 outgoing (SO_SNDBUF socket option).
	# 0 is system default.  Use 4m to handle spikes on very busy servers.
	so-sndbuf: 1m

	# use SO_REUSEPORT to distribute queries over threads.
	# at extreme load it could be better to turn it off to distribute even.
	# so-reuseport: yes

	# use IP_TRANSPARENT so the interface: addresses can be non-local
	# and you can config non-existing IPs that are going to work later on
	# (uses IP_BINDANY on FreeBSD).
	# ip-transparent: no

	# use IP_FREEBIND so the interface: addresses can be non-local
	# and you can bind to nonexisting IPs and interfaces that are down.
	# Linux only.  On Linux you also have ip-transparent that is similar.
	# ip-freebind: no

	# the value of the Differentiated Services Codepoint (DSCP)
	# in the differentiated services field (DS) of the outgoing
	# IP packets
	# ip-dscp: 0

	# EDNS reassembly buffer to advertise to UDP peers (the actual buffer
	# is set with msg-buffer-size).
	# edns-buffer-size: 1232

	# Maximum UDP response size (not applied to TCP response).
	# Suggested values are 512 to 4096. Default is 4096. 65536 disables it.
	# max-udp-size: 4096

	# max memory to use for stream(tcp and tls) waiting result buffers.
	# stream-wait-size: 4m

	# buffer size for handling DNS data. No messages larger than this
	# size can be sent or received, by UDP or TCP. In bytes.
	# msg-buffer-size: 65552

	# the amount of memory to use for the message cache.
	# plain value in bytes or you can append k, m or G. default is "4Mb".
	msg-cache-size: 128m

	# the number of slabs to use for the message cache.
	# the number of slabs must be a power of 2.
	# more slabs reduce lock contention, but fragment memory usage.
	# msg-cache-slabs: 4

	# the number of queries that a thread gets to service.
	# num-queries-per-thread: 1024

	# if very busy, 50% queries run to completion, 50% get timeout in msec
	# jostle-timeout: 200

	# msec to wait before close of port on timeout UDP. 0 disables.
	# delay-close: 0

	# perform connect for UDP sockets to mitigate ICMP side channel.
	# udp-connect: yes

	# The number of retries, per upstream nameserver in a delegation, when
	# a throwaway response (also timeouts) is received.
	# outbound-msg-retry: 5

	# Hard limit on the number of outgoing queries Unbound will make while
	# resolving a name, making sure large NS sets do not loop.
	# It resets on query restarts (e.g., CNAME) and referrals.
	# max-sent-count: 32

	# Hard limit on the number of times Unbound is allowed to restart a
	# query upon encountering a CNAME record.
	# max-query-restarts: 11

	# msec for waiting for an unknown server to reply.  Increase if you
	# are behind a slow satellite link, to eg. 1128.
	# unknown-server-time-limit: 376

	# the amount of memory to use for the RRset cache.
	# plain value in bytes or you can append k, m or G. default is "4Mb".
	rrset-cache-size: 256m

	# the number of slabs to use for the RRset cache.
	# the number of slabs must be a power of 2.
	# more slabs reduce lock contention, but fragment memory usage.
	# rrset-cache-slabs: 4

	# the time to live (TTL) value lower bound, in seconds. Default 0.
	# If more than an hour could easily give trouble due to stale data.
	# cache-min-ttl: 0

	# the time to live (TTL) value cap for RRsets and messages in the
	# cache. Items are not cached for longer. In seconds.
	# cache-max-ttl: 86400

	# the time to live (TTL) value cap for negative responses in the cache
	# cache-max-negative-ttl: 3600

	# the time to live (TTL) value for cached roundtrip times, lameness and
	# EDNS version information for hosts. In seconds.
	# infra-host-ttl: 900

	# minimum wait time for responses, increase if uplink is long. In msec.
	# infra-cache-min-rtt: 50

	# maximum wait time for responses. In msec.
	# infra-cache-max-rtt: 120000

	# enable to make server probe down hosts more frequently.
	# infra-keep-probing: no

	# the number of slabs to use for the Infrastructure cache.
	# the number of slabs must be a power of 2.
	# more slabs reduce lock contention, but fragment memory usage.
	# infra-cache-slabs: 4

	# the maximum number of hosts that are cached (roundtrip, EDNS, lame).
	# infra-cache-numhosts: 10000

	# define a number of tags here, use with local-zone, access-control,
	# interface-*.
	# repeat the define-tag statement to add additional tags.
	# define-tag: "tag1 tag2 tag3"

	# Enable IPv4, "yes" or "no".
	do-ip4: yes

	# Enable IPv6, "yes" or "no".
	# do-ip6: yes

	# Enable UDP, "yes" or "no".
	do-udp: yes

	# Enable TCP, "yes" or "no".
	do-tcp: yes

	# upstream connections use TCP only (and no UDP), "yes" or "no"
	# useful for tunneling scenarios, default no.
	# tcp-upstream: no

	# upstream connections also use UDP (even if do-udp is no).
	# useful if if you want UDP upstream, but don't provide UDP downstream.
	# udp-upstream-without-downstream: no

	# Maximum segment size (MSS) of TCP socket on which the server
	# responds to queries. Default is 0, system default MSS.
	# tcp-mss: 0

	# Maximum segment size (MSS) of TCP socket for outgoing queries.
	# Default is 0, system default MSS.
	# outgoing-tcp-mss: 0

	# Idle TCP timeout, connection closed in milliseconds
	# tcp-idle-timeout: 30000

	# Enable EDNS TCP keepalive option.
	# edns-tcp-keepalive: no

	# Timeout for EDNS TCP keepalive, in msec.
	# edns-tcp-keepalive-timeout: 120000

	# Use systemd socket activation for UDP, TCP, and control sockets.
	# use-systemd: no

	# Detach from the terminal, run in background, "yes" or "no".
	# Set the value to "no" when Unbound runs as systemd service.
	# do-daemonize: yes

	# control which clients are allowed to make (recursive) queries
	# to this server. Specify classless netblocks with /size and action.
	# By default everything is refused, except for localhost.
	# Choose deny (drop message), refuse (polite error reply),
	# allow (recursive ok), allow_setrd (recursive ok, rd bit is forced on),
	# allow_snoop (recursive and nonrecursive ok)
	# deny_non_local (drop queries unless can be answered from local-data)
	# refuse_non_local (like deny_non_local but polite error reply).
	access-control: 192.168.5.0/24 allow
	access-control: 127.0.0.0/8 allow	
	# access-control: ::ffff:127.0.0.1 allow

	# tag access-control with list of tags (in "" with spaces between)
	# Clients using this access control element use localzones that
	# are tagged with one of these tags.
	# access-control-tag: 192.0.2.0/24 "tag2 tag3"

	# set action for particular tag for given access control element.
	# if you have multiple tag values, the tag used to lookup the action
	# is the first tag match between access-control-tag and local-zone-tag
	# where "first" comes from the order of the define-tag values.
	# access-control-tag-action: 192.0.2.0/24 tag3 refuse

	# set redirect data for particular tag for access control element
	# access-control-tag-data: 192.0.2.0/24 tag2 "A 127.0.0.1"

	# Set view for access control element
	# access-control-view: 192.0.2.0/24 viewname

	# Similar to 'access-control:' but for interfaces.
	# Control which listening interfaces are allowed to accept (recursive)
	# queries for this server.
	# The specified interfaces should be the same as the ones specified in
	# 'interface:' followed by the action.
	# The actions are the same as 'access-control:' above.
	# By default all the interfaces configured are refused.
	# Note: any 'access-control*:' setting overrides all 'interface-*:'
	# settings for targeted clients.
	# interface-action: 192.0.2.153 allow
	# interface-action: 192.0.2.154 allow
	# interface-action: 192.0.2.154@5003 allow
	# interface-action: 2001:DB8::5 allow
	# interface-action: eth0@5003 allow

	# Similar to 'access-control-tag:' but for interfaces.
	# Tag interfaces with a list of tags (in "" with spaces between).
	# Interfaces using these tags use localzones that are tagged with one
	# of these tags.
	# The specified interfaces should be the same as the ones specified in
	# 'interface:' followed by the list of tags.
	# Note: any 'access-control*:' setting overrides all 'interface-*:'
	# settings for targeted clients.
	# interface-tag: eth0@5003 "tag2 tag3"

	# Similar to 'access-control-tag-action:' but for interfaces.
	# Set action for particular tag for a given interface element.
	# If you have multiple tag values, the tag used to lookup the action
	# is the first tag match between interface-tag and local-zone-tag
	# where "first" comes from the order of the define-tag values.
	# The specified interfaces should be the same as the ones specified in
	# 'interface:' followed by the tag and action.
	# Note: any 'access-control*:' setting overrides all 'interface-*:'
	# settings for targeted clients.
	# interface-tag-action: eth0@5003 tag3 refuse

	# Similar to 'access-control-tag-data:' but for interfaces.
	# Set redirect data for a particular tag for an interface element.
	# The specified interfaces should be the same as the ones specified in
	# 'interface:' followed by the tag and the redirect data.
	# Note: any 'access-control*:' setting overrides all 'interface-*:'
	# settings for targeted clients.
	# interface-tag-data: eth0@5003 tag2 "A 127.0.0.1"

	# Similar to 'access-control-view:' but for interfaces.
	# Set view for an interface element.
	# The specified interfaces should be the same as the ones specified in
	# 'interface:' followed by the view name.
	# Note: any 'access-control*:' setting overrides all 'interface-*:'
	# settings for targeted clients.
	# interface-view: eth0@5003 viewname

	# if given, a chroot(2) is done to the given directory.
	# i.e. you can chroot to the working directory, for example,
	# for extra security, but make sure all files are in that directory.
	#
	# If chroot is enabled, you should pass the configfile (from the
	# commandline) as a full path from the original root. After the
	# chroot has been performed the now defunct portion of the config
	# file path is removed to be able to reread the config after a reload.
	#
	# All other file paths (working dir, logfile, roothints, and
	# key files) can be specified in several ways:
	# 	o as an absolute path relative to the new root.
	# 	o as a relative path to the working directory.
	# 	o as an absolute path relative to the original root.
	# In the last case the path is adjusted to remove the unused portion.
	#
	# The pid file can be absolute and outside of the chroot, it is
	# written just prior to performing the chroot and dropping permissions.
	#
	# Additionally, Unbound may need to access /dev/urandom (for entropy).
	# How to do this is specific to your OS.
	#
	# If you give "" no chroot is performed. The path must not end in a /.
	chroot: "/usr/local/etc/unbound"

	# if given, user privileges are dropped (after binding port),
	# and the given username is assumed. Default is user "unbound".
	# If you give "" no privileges are dropped.
	username: "unbound"

	# the working directory. The relative files in this config are
	# relative to this directory. If you give "" the working directory
	# is not changed.
	# If you give a server: directory: dir before include: file statements
	# then those includes can be relative to the working directory.
	directory: "/usr/local/etc/unbound"

	# the log file, "" means log to stderr.
	# Use of this option sets use-syslog to "no".
	logfile: "/usr/local/etc/unbound/log/unbound.log"

	# Log to syslog(3) if yes. The log facility LOG_DAEMON is used to
	# log to. If yes, it overrides the logfile.
	# use-syslog: yes

	# Log identity to report. if empty, defaults to the name of argv[0]
	# (usually "unbound").
	# log-identity: ""

	# print UTC timestamp in ascii to logfile, default is epoch in seconds.
	log-time-ascii: yes

	# print one line with time, IP, name, type, class for every query.
	# log-queries: no

	# print one line per reply, with time, IP, name, type, class, rcode,
	# timetoresolve, fromcache and responsesize.
	# log-replies: no

	# log with tag 'query' and 'reply' instead of 'info' for
	# filtering log-queries and log-replies from the log.
	# log-tag-queryreply: no

	# log the local-zone actions, like local-zone type inform is enabled
	# also for the other local zone types.
	# log-local-actions: no

	# print log lines that say why queries return SERVFAIL to clients.
	# log-servfail: no

	# the pid file. Can be an absolute path outside of chroot/work dir.
	pidfile: "/usr/local/etc/unbound/unbound.pid"

	# file to read root hints from.
	# get one from https://www.internic.net/domain/named.cache
	root-hints: "/usr/local/etc/unbound/root.hints"

	# enable to not answer id.server and hostname.bind queries.
	hide-identity: yes

	# enable to not answer version.server and version.bind queries.
	hide-version: yes

	# enable to not answer trustanchor.unbound queries.
	# hide-trustanchor: no

	# enable to not set the User-Agent HTTP header.
	# hide-http-user-agent: no

	# the identity to report. Leave "" or default to return hostname.
	# identity: ""

	# the version to report. Leave "" or default to return package version.
	# version: ""

	# NSID identity (hex string, or "ascii_somestring"). default disabled.
	# nsid: "aabbccdd"

	# User-Agent HTTP header to use. Leave "" or default to use package name
	# and version.
	# http-user-agent: ""

	# the target fetch policy.
	# series of integers describing the policy per dependency depth.
	# The number of values in the list determines the maximum dependency
	# depth the recursor will pursue before giving up. Each integer means:
	# 	-1 : fetch all targets opportunistically,
	# 	0: fetch on demand,
	#	positive value: fetch that many targets opportunistically.
	# Enclose the list of numbers between quotes ("").
	# target-fetch-policy: "3 2 1 0 0"

	# Harden against very small EDNS buffer sizes.
	# harden-short-bufsize: yes

	# Harden against unseemly large queries.
	# harden-large-queries: no

	# Harden against out of zone rrsets, to avoid spoofing attempts.
	harden-glue: yes

	# Harden against receiving dnssec-stripped data. If you turn it
	# off, failing to validate dnskey data for a trustanchor will
	# trigger insecure mode for that zone (like without a trustanchor).
	# Default on, which insists on dnssec data for trust-anchored zones.
	harden-dnssec-stripped: yes

	# Harden against queries that fall under dnssec-signed nxdomain names.
	# harden-below-nxdomain: yes

	# Harden the referral path by performing additional queries for
	# infrastructure data.  Validates the replies (if possible).
	# Default off, because the lookups burden the server.  Experimental
	# implementation of draft-wijngaards-dnsext-resolver-side-mitigation.
	# harden-referral-path: no

	# Harden against algorithm downgrade when multiple algorithms are
	# advertised in the DS record.  If no, allows the weakest algorithm
	# to validate the zone.
	# harden-algo-downgrade: no

	# Sent minimum amount of information to upstream servers to enhance
	# privacy. Only sent minimum required labels of the QNAME and set QTYPE
	# to A when possible.
	# qname-minimisation: yes

	# QNAME minimisation in strict mode. Do not fall-back to sending full
	# QNAME to potentially broken nameservers. A lot of domains will not be
	# resolvable when this option in enabled.
	# This option only has effect when qname-minimisation is enabled.
	# qname-minimisation-strict: no

	# Aggressive NSEC uses the DNSSEC NSEC chain to synthesize NXDOMAIN
	# and other denials, using information from previous NXDOMAINs answers.
	# aggressive-nsec: yes

	# Use 0x20-encoded random bits in the query to foil spoof attempts.
	# This feature is an experimental implementation of draft dns-0x20.
	# use-caps-for-id: no

	# Domains (and domains in them) without support for dns-0x20 and
	# the fallback fails because they keep sending different answers.
	# caps-exempt: "licdn.com"
	# caps-exempt: "senderbase.org"

	# Enforce privacy of these addresses. Strips them away from answers.
	# It may cause DNSSEC validation to additionally mark it as bogus.
	# Protects against 'DNS Rebinding' (uses browser as network proxy).
	# Only 'private-domain' and 'local-data' names are allowed to have
	# these private addresses. No default.
	private-address: 10.0.0.0/8
	private-address: 172.16.0.0/12
	private-address: 192.168.0.0/16
	private-address: 169.254.0.0/16
	private-address: fd00::/8
	private-address: fe80::/10
	private-address: ::ffff:0:0/96

	# Allow the domain (and its subdomains) to contain private addresses.
	# local-data statements are allowed to contain private addresses too.
	private-domain: "unixexplore.com"

	# If nonzero, unwanted replies are not only reported in statistics,
	# but also a running total is kept per thread. If it reaches the
	# threshold, a warning is printed and a defensive action is taken,
	# the cache is cleared to flush potential poison out of it.
	# A suggested value is 10000000, the default is 0 (turned off).
	# unwanted-reply-threshold: 0

	# Do not query the following addresses. No DNS queries are sent there.
	# List one address per entry. List classless netblocks with /size,
	# do-not-query-address: 127.0.0.1/8
	# do-not-query-address: ::1

	# if yes, the above default do-not-query-address entries are present.
	# if no, localhost can be queried (for testing and debugging).
	# do-not-query-localhost: yes

	# if yes, perform prefetching of almost expired message cache entries.
	# prefetch: no

	# if yes, perform key lookups adjacent to normal lookups.
	# prefetch-key: no

	# deny queries of type ANY with an empty response.
	# deny-any: no

	# if yes, Unbound rotates RRSet order in response.
	# rrset-roundrobin: yes

	# if yes, Unbound doesn't insert authority/additional sections
	# into response messages when those sections are not required.
	# minimal-responses: yes

	# true to disable DNSSEC lameness check in iterator.
	# disable-dnssec-lame-check: no

	# module configuration of the server. A string with identifiers
	# separated by spaces. Syntax: "[dns64] [validator] iterator"
	# most modules have to be listed at the beginning of the line,
	# except cachedb(just before iterator), and python (at the beginning,
	# or, just before the iterator).
	# module-config: "validator iterator"

	# File with trusted keys, kept uptodate using RFC5011 probes,
	# initial file like trust-anchor-file, then it stores metadata.
	# Use several entries, one per domain name, to track multiple zones.
	#
	# If you want to perform DNSSEC validation, run unbound-anchor before
	# you start Unbound (i.e. in the system boot scripts).
	# And then enable the auto-trust-anchor-file config item.
	# Please note usage of unbound-anchor root anchor is at your own risk
	# and under the terms of our LICENSE (see that file in the source).
	auto-trust-anchor-file: "/usr/local/etc/unbound/root.key"

	# trust anchor signaling sends a RFC8145 key tag query after priming.
	# trust-anchor-signaling: yes

	# Root key trust anchor sentinel (draft-ietf-dnsop-kskroll-sentinel)
	# root-key-sentinel: yes

	# File with trusted keys for validation. Specify more than one file
	# with several entries, one file per entry.
	# Zone file format, with DS and DNSKEY entries.
	# Note this gets out of date, use auto-trust-anchor-file please.
	# trust-anchor-file: ""

	# Trusted key for validation. DS or DNSKEY. specify the RR on a
	# single line, surrounded by "". TTL is ignored. class is IN default.
	# Note this gets out of date, use auto-trust-anchor-file please.
	# (These examples are from August 2007 and may not be valid anymore).
	# trust-anchor: "nlnetlabs.nl. DNSKEY 257 3 5 AQPzzTWMz8qSWIQlfRnPckx2BiVmkVN6LPupO3mbz7FhLSnm26n6iG9N Lby97Ji453aWZY3M5/xJBSOS2vWtco2t8C0+xeO1bc/d6ZTy32DHchpW 6rDH1vp86Ll+ha0tmwyy9QP7y2bVw5zSbFCrefk8qCUBgfHm9bHzMG1U BYtEIQ=="
	# trust-anchor: "jelte.nlnetlabs.nl. DS 42860 5 1 14D739EB566D2B1A5E216A0BA4D17FA9B038BE4A"

	# File with trusted keys for validation. Specify more than one file
	# with several entries, one file per entry. Like trust-anchor-file
	# but has a different file format. Format is BIND-9 style format,
	# the trusted-keys { name flag proto algo "key"; }; clauses are read.
	# you need external update procedures to track changes in keys.
	# trusted-keys-file: ""

	# Ignore chain of trust. Domain is treated as insecure.
	domain-insecure: "unixexplore.com"
	domain-insecure: "9.168.192.in-addr.arpa"

	# Override the date for validation with a specific fixed date.
	# Do not set this unless you are debugging signature inception
	# and expiration. "" or "0" turns the feature off. -1 ignores date.
	# val-override-date: ""

	# The time to live for bogus data, rrsets and messages. This avoids
	# some of the revalidation, until the time interval expires. in secs.
	# val-bogus-ttl: 60

	# The signature inception and expiration dates are allowed to be off
	# by 10% of the signature lifetime (expir-incep) from our local clock.
	# This leeway is capped with a minimum and a maximum.  In seconds.
	# val-sig-skew-min: 3600
	# val-sig-skew-max: 86400

	# The maximum number the validator should restart validation with
	# another authority in case of failed validation.
	# val-max-restart: 5

	# Should additional section of secure message also be kept clean of
	# unsecure data. Useful to shield the users of this validator from
	# potential bogus data in the additional section. All unsigned data
	# in the additional section is removed from secure messages.
	val-clean-additional: yes

	# Turn permissive mode on to permit bogus messages. Thus, messages
	# for which security checks failed will be returned to clients,
	# instead of SERVFAIL. It still performs the security checks, which
	# result in interesting log files and possibly the AD bit in
	# replies if the message is found secure. The default is off.
	# val-permissive-mode: no

	# Ignore the CD flag in incoming queries and refuse them bogus data.
	# Enable it if the only clients of Unbound are legacy servers (w2008)
	# that set CD but cannot validate themselves.
	# ignore-cd-flag: no

	# Serve expired responses from cache, with serve-expired-reply-ttl in
	# the response, and then attempt to fetch the data afresh.
	# serve-expired: no
	#
	# Limit serving of expired responses to configured seconds after
	# expiration. 0 disables the limit.
	# serve-expired-ttl: 0
	#
	# Set the TTL of expired records to the serve-expired-ttl value after a
	# failed attempt to retrieve the record from upstream. This makes sure
	# that the expired records will be served as long as there are queries
	# for it.
	# serve-expired-ttl-reset: no
	#
	# TTL value to use when replying with expired data.
	# serve-expired-reply-ttl: 30
	#
	# Time in milliseconds before replying to the client with expired data.
	# This essentially enables the serve-stale behavior as specified in
	# RFC 8767 that first tries to resolve before
	# immediately responding with expired data.  0 disables this behavior.
	# A recommended value is 1800.
	# serve-expired-client-timeout: 0

	# Return the original TTL as received from the upstream name server rather
	# than the decrementing TTL as stored in the cache.  Enabling this feature
	# does not impact cache expiry, it only changes the TTL Unbound embeds in
	# responses to queries. Note that enabling this feature implicitly disables
	# enforcement of the configured minimum and maximum TTL.
	# serve-original-ttl: no

	# Have the validator log failed validations for your diagnosis.
	# 0: off. 1: A line per failed user query. 2: With reason and bad IP.
	# val-log-level: 0

	# It is possible to configure NSEC3 maximum iteration counts per
	# keysize. Keep this table very short, as linear search is done.
	# A message with an NSEC3 with larger count is marked insecure.
	# List in ascending order the keysize and count values.
	# val-nsec3-keysize-iterations: "1024 150 2048 150 4096 150"

	# if enabled, ZONEMD verification failures do not block the zone.
	# zonemd-permissive-mode: no

	# instruct the auto-trust-anchor-file probing to add anchors after ttl.
	# add-holddown: 2592000 # 30 days

	# instruct the auto-trust-anchor-file probing to del anchors after ttl.
	# del-holddown: 2592000 # 30 days

	# auto-trust-anchor-file probing removes missing anchors after ttl.
	# If the value 0 is given, missing anchors are not removed.
	# keep-missing: 31622400 # 366 days

	# debug option that allows very small holddown times for key rollover,
	# otherwise the RFC mandates probe intervals must be at least 1 hour.
	# permit-small-holddown: no

	# the amount of memory to use for the key cache.
	# plain value in bytes or you can append k, m or G. default is "4Mb".
	# key-cache-size: 4m

	# the number of slabs to use for the key cache.
	# the number of slabs must be a power of 2.
	# more slabs reduce lock contention, but fragment memory usage.
	# key-cache-slabs: 4

	# the amount of memory to use for the negative cache.
	# plain value in bytes or you can append k, m or G. default is "1Mb".
	# neg-cache-size: 1m

	# By default, for a number of zones a small default 'nothing here'
	# reply is built-in.  Query traffic is thus blocked.  If you
	# wish to serve such zone you can unblock them by uncommenting one
	# of the nodefault statements below.
	# You may also have to use domain-insecure: zone to make DNSSEC work,
	# unless you have your own trust anchors for this zone.
	# local-zone: "localhost." nodefault
	# local-zone: "127.in-addr.arpa." nodefault
	# local-zone: "1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.ip6.arpa." nodefault
	# local-zone: "home.arpa." nodefault
	# local-zone: "onion." nodefault
	# local-zone: "test." nodefault
	# local-zone: "invalid." nodefault
	# local-zone: "10.in-addr.arpa." nodefault
	# local-zone: "16.172.in-addr.arpa." nodefault
	# local-zone: "17.172.in-addr.arpa." nodefault
	# local-zone: "18.172.in-addr.arpa." nodefault
	# local-zone: "19.172.in-addr.arpa." nodefault
	# local-zone: "20.172.in-addr.arpa." nodefault
	# local-zone: "21.172.in-addr.arpa." nodefault
	# local-zone: "22.172.in-addr.arpa." nodefault
	# local-zone: "23.172.in-addr.arpa." nodefault
	# local-zone: "24.172.in-addr.arpa." nodefault
	# local-zone: "25.172.in-addr.arpa." nodefault
	# local-zone: "26.172.in-addr.arpa." nodefault
	# local-zone: "27.172.in-addr.arpa." nodefault
	# local-zone: "28.172.in-addr.arpa." nodefault
	# local-zone: "29.172.in-addr.arpa." nodefault
	# local-zone: "30.172.in-addr.arpa." nodefault
	# local-zone: "31.172.in-addr.arpa." nodefault
	local-zone: "168.192.in-addr.arpa." nodefault
	# local-zone: "0.in-addr.arpa." nodefault
	# local-zone: "254.169.in-addr.arpa." nodefault
	# local-zone: "2.0.192.in-addr.arpa." nodefault
	# local-zone: "100.51.198.in-addr.arpa." nodefault
	# local-zone: "113.0.203.in-addr.arpa." nodefault
	# local-zone: "255.255.255.255.in-addr.arpa." nodefault
	# local-zone: "0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.ip6.arpa." nodefault
	# local-zone: "d.f.ip6.arpa." nodefault
	# local-zone: "8.e.f.ip6.arpa." nodefault
	# local-zone: "9.e.f.ip6.arpa." nodefault
	# local-zone: "a.e.f.ip6.arpa." nodefault
	# local-zone: "b.e.f.ip6.arpa." nodefault
	# local-zone: "8.b.d.0.1.0.0.2.ip6.arpa." nodefault
	# And for 64.100.in-addr.arpa. to 127.100.in-addr.arpa.

	# Add example.com into ipset
	# local-zone: "example.com" ipset

	# If Unbound is running service for the local host then it is useful
	# to perform lan-wide lookups to the upstream, and unblock the
	# long list of local-zones above.  If this Unbound is a dns server
	# for a network of computers, disabled is better and stops information
	# leakage of local lan information.
	# unblock-lan-zones: no

	# The insecure-lan-zones option disables validation for
	# these zones, as if they were all listed as domain-insecure.
	# insecure-lan-zones: no

	# a number of locally served zones can be configured.
	# 	local-zone: <zone> <type>
	# 	local-data: "<resource record string>"
	# o deny serves local data (if any), else, drops queries.
	# o refuse serves local data (if any), else, replies with error.
	# o static serves local data, else, nxdomain or nodata answer.
	# o transparent gives local data, but resolves normally for other names
	# o redirect serves the zone data for any subdomain in the zone.
	# o nodefault can be used to normally resolve AS112 zones.
	# o typetransparent resolves normally for other types and other names
	# o inform acts like transparent, but logs client IP address
	# o inform_deny drops queries and logs client IP address
	# o inform_redirect redirects queries and logs client IP address
	# o always_transparent, always_refuse, always_nxdomain, always_nodata,
	#   always_deny resolve in that way but ignore local data for
	#   that name
	# o always_null returns 0.0.0.0 or ::0 for any name in the zone.
	# o noview breaks out of that view towards global local-zones.
	#
	# defaults are localhost address, reverse for 127.0.0.1 and ::1
	# and nxdomain for AS112 zones. If you configure one of these zones
	# the default content is omitted, or you can omit it with 'nodefault'.
	#
	# If you configure local-data without specifying local-zone, by
	# default a transparent local-zone is created for the data.
	#
	# You can add locally served data with
	# local-zone: "local." static
	# local-data: "mycomputer.local. IN A 192.0.2.51"
	# local-data: 'mytext.local TXT "content of text record"'
	#
	# You can override certain queries with
	# local-data: "adserver.example.com A 127.0.0.1"
	#
	# You can redirect a domain to a fixed address with
	# (this makes example.com, www.example.com, etc, all go to 192.0.2.3)
	# local-zone: "example.com" redirect
	# local-data: "example.com A 192.0.2.3"
	#
	# Shorthand to make PTR records, "IPv4 name" or "IPv6 name".
	# You can also add PTR records using local-data directly, but then
	# you need to do the reverse notation yourself.
	# local-data-ptr: "192.0.2.3 www.example.com"

	# tag a localzone with a list of tag names (in "" with spaces between)
	# local-zone-tag: "example.com" "tag2 tag3"

	# add a netblock specific override to a localzone, with zone type
	# local-zone-override: "example.com" 192.0.2.0/24 refuse

	# service clients over TLS (on the TCP sockets) with plain DNS inside
	# the TLS stream, and over HTTPS using HTTP/2 as specified in RFC8484.
	# Give the certificate to use and private key.
	# default is "" (disabled).  requires restart to take effect.
	tls-service-key: "/etc/ssl/unbound/mydomain.key"
	tls-service-pem: "/etc/ssl/unbound/mydomain.pem"
	tls-port: 853
	# https-port: 443

	# cipher setting for TLSv1.2
	# tls-ciphers: "DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-SHA256:DHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256"
	# cipher setting for TLSv1.3
	# tls-ciphersuites: "TLS_AES_128_GCM_SHA256:TLS_AES_128_CCM_8_SHA256:TLS_AES_128_CCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256"

	# Pad responses to padded queries received over TLS
	# pad-responses: yes

	# Padded responses will be padded to the closest multiple of this size.
	# pad-responses-block-size: 468

	# Use the SNI extension for TLS connections.  Default is yes.
	# Changing the value requires a reload.
	# tls-use-sni: yes

	# Add the secret file for TLS Session Ticket.
	# Secret file must be 80 bytes of random data.
	# First key use to encrypt and decrypt TLS session tickets.
	# Other keys use to decrypt only.
	# requires restart to take effect.
	# tls-session-ticket-keys: "path/to/secret_file1"
	# tls-session-ticket-keys: "path/to/secret_file2"

	# request upstream over TLS (with plain DNS inside the TLS stream).
	# Default is no.  Can be turned on and off with unbound-control.
	# tls-upstream: no

	# Certificates used to authenticate connections made upstream.
	# tls-cert-bundle: ""

	# Add system certs to the cert bundle, from the Windows Cert Store
	# tls-win-cert: no
	# and on other systems, the default openssl certificates
	# tls-system-cert: no

	# Pad queries over TLS upstreams
	# pad-queries: yes

	# Padded queries will be padded to the closest multiple of this size.
	# pad-queries-block-size: 128

	# Also serve tls on these port numbers (eg. 443, ...), by listing
	# tls-additional-port: portno for each of the port numbers.

	# HTTP endpoint to provide DNS-over-HTTPS service on.
	# http-endpoint: "/dns-query"

	# HTTP/2 SETTINGS_MAX_CONCURRENT_STREAMS value to use.
	# http-max-streams: 100

	# Maximum number of bytes used for all HTTP/2 query buffers.
	# http-query-buffer-size: 4m

	# Maximum number of bytes used for all HTTP/2 response buffers.
	# http-response-buffer-size: 4m

	# Set TCP_NODELAY socket option on sockets used for DNS-over-HTTPS
	# service.
	# http-nodelay: yes

	# Disable TLS for DNS-over-HTTP downstream service.
	# http-notls-downstream: no

	# The interfaces that use these listed port numbers will support and
	# expect PROXYv2. For UDP and TCP/TLS interfaces.
	# proxy-protocol-port: portno for each of the port numbers.

	# DNS64 prefix. Must be specified when DNS64 is use.
	# Enable dns64 in module-config.  Used to synthesize IPv6 from IPv4.
	# dns64-prefix: 64:ff9b::0/96

	# DNS64 ignore AAAA records for these domains and use A instead.
	# dns64-ignore-aaaa: "example.com"

	# ratelimit for uncached, new queries, this limits recursion effort.
	# ratelimiting is experimental, and may help against randomqueryflood.
	# if 0(default) it is disabled, otherwise state qps allowed per zone.
	# ratelimit: 0

	# ratelimits are tracked in a cache, size in bytes of cache (or k,m).
	# ratelimit-size: 4m
	# ratelimit cache slabs, reduces lock contention if equal to cpucount.
	# ratelimit-slabs: 4

	# 0 blocks when ratelimited, otherwise let 1/xth traffic through
	# ratelimit-factor: 10

	# Aggressive rate limit when the limit is reached and until demand has
	# decreased in a 2 second rate window.
	# ratelimit-backoff: no

	# override the ratelimit for a specific domain name.
	# give this setting multiple times to have multiple overrides.
	# ratelimit-for-domain: example.com 1000
	# override the ratelimits for all domains below a domain name
	# can give this multiple times, the name closest to the zone is used.
	# ratelimit-below-domain: com 1000

	# global query ratelimit for all ip addresses.
	# feature is experimental.
	# if 0(default) it is disabled, otherwise states qps allowed per ip address
	# ip-ratelimit: 0

	# ip ratelimits are tracked in a cache, size in bytes of cache (or k,m).
	# ip-ratelimit-size: 4m
	# ip ratelimit cache slabs, reduces lock contention if equal to cpucount.
	# ip-ratelimit-slabs: 4

	# 0 blocks when ip is ratelimited, otherwise let 1/xth traffic through
	# ip-ratelimit-factor: 10

	# Aggressive rate limit when the limit is reached and until demand has
	# decreased in a 2 second rate window.
	# ip-ratelimit-backoff: no

	# Limit the number of connections simultaneous from a netblock
	# tcp-connection-limit: 192.0.2.0/24 12

	# select from the fastest servers this many times out of 1000. 0 means
	# the fast server select is disabled. prefetches are not sped up.
	# fast-server-permil: 0
	# the number of servers that will be used in the fast server selection.
	# fast-server-num: 3

	# Enable to attach Extended DNS Error codes (RFC8914) to responses.
	# ede: no

	# Enable to attach an Extended DNS Error (RFC8914) Code 3 - Stale
	# Answer as EDNS0 option to expired responses.
	# Note that the ede option above needs to be enabled for this to work.
	# ede-serve-expired: no

	# Specific options for ipsecmod. Unbound needs to be configured with
	# --enable-ipsecmod for these to take effect.
	#
	# Enable or disable ipsecmod (it still needs to be defined in
	# module-config above). Can be used when ipsecmod needs to be
	# enabled/disabled via remote-control(below).
	# ipsecmod-enabled: yes
	#
	# Path to executable external hook. It must be defined when ipsecmod is
	# listed in module-config (above).
	# ipsecmod-hook: "./my_executable"
	#
	# When enabled Unbound will reply with SERVFAIL if the return value of
	# the ipsecmod-hook is not 0.
	# ipsecmod-strict: no
	#
	# Maximum time to live (TTL) for cached A/AAAA records with IPSECKEY.
	# ipsecmod-max-ttl: 3600
	#
	# Reply with A/AAAA even if the relevant IPSECKEY is bogus. Mainly used for
	# testing.
	# ipsecmod-ignore-bogus: no
	#
	# Domains for which ipsecmod will be triggered. If not defined (default)
	# all domains are treated as being allowed.
	# ipsecmod-allow: "example.com"
	# ipsecmod-allow: "nlnetlabs.nl"

	# Timeout for REUSE entries in milliseconds.
	# tcp-reuse-timeout: 60000
	# Max number of queries on a reuse connection.
	# max-reuse-tcp-queries: 200
	# Timeout in milliseconds for TCP queries to auth servers.
	# tcp-auth-query-timeout: 3000


# Python config section. To enable:
# o use --with-pythonmodule to configure before compiling.
# o list python in the module-config string (above) to enable.
#   It can be at the start, it gets validated results, or just before
#   the iterator and process before DNSSEC validation.
# o and give a python-script to run.
python:
	# Script file to load
	# python-script: "/usr/local/etc/unbound/ubmodule-tst.py"

# Dynamic library config section. To enable:
# o use --with-dynlibmodule to configure before compiling.
# o list dynlib in the module-config string (above) to enable.
#   It can be placed anywhere, the dynlib module is only a very thin wrapper
#   to load modules dynamically.
# o and give a dynlib-file to run. If more than one dynlib entry is listed in
#   the module-config then you need one dynlib-file per instance.
dynlib:
	# Script file to load
	# dynlib-file: "/usr/local/etc/unbound/dynlib.so"

# Remote control config section.
remote-control:
	# Enable remote control with unbound-control(8) here.
	# set up the keys and certificates with unbound-control-setup.
	control-enable: yes

	# what interfaces are listened to for remote control.
	# give 0.0.0.0 and ::0 to listen to all interfaces.
	# set to an absolute path to use a unix local name pipe, certificates
	# are not used for that, so key and cert files need not be present.
	control-interface: 127.0.0.1
	# control-interface: ::1

	# port number for remote control operations.
	control-port: 8953

	# for localhost, you can disable use of TLS by setting this to "no"
	# For local sockets this option is ignored, and TLS is not used.
	# control-use-cert: "yes"

	# Unbound server key file.
	server-key-file: "/usr/local/etc/unbound/unbound_server.key"

	# Unbound server certificate file.
	server-cert-file: "/usr/local/etc/unbound/unbound_server.pem"

	# unbound-control key file.
	control-key-file: "/usr/local/etc/unbound/unbound_control.key"

	# unbound-control certificate file.
	control-cert-file: "/usr/local/etc/unbound/unbound_control.pem"

# Stub zones.
# Create entries like below, to make all queries for 'example.com' and
# 'example.org' go to the given list of nameservers. list zero or more
# nameservers by hostname or by ipaddress. If you set stub-prime to yes,
# the list is treated as priming hints (default is no).
# With stub-first yes, it attempts without the stub if it fails.
# Consider adding domain-insecure: name and local-zone: name nodefault
# to the server: section if the stub is a locally served zone.
# stub-zone:
#	name: "example.com"
#	stub-addr: 192.0.2.68
#	stub-prime: no
#	stub-first: no
#	stub-tcp-upstream: no
#	stub-tls-upstream: no
#	stub-no-cache: no
# stub-zone:
#	name: "example.org"
#	stub-host: ns.example.com.

# Forward zones
# Create entries like below, to make all queries for 'example.com' and
# 'example.org' go to the given list of servers. These servers have to handle
# recursion to other nameservers. List zero or more nameservers by hostname
# or by ipaddress. Use an entry with name "." to forward all queries.
# If you enable forward-first, it attempts without the forward if it fails.
# forward-zone:
# 	name: "example.com"
# 	forward-addr: 192.0.2.68
# 	forward-addr: 192.0.2.73@5355  # forward to port 5355.
# 	forward-first: no
# 	forward-tcp-upstream: no
# 	forward-tls-upstream: no
#	forward-no-cache: no
# forward-zone:
# 	name: "example.org"
# 	forward-host: fwd.example.com

forward-zone:
name: "."
forward-first: no
forward-ssl-upstream: yes
forward-addr: 1.1.1.1@853
forward-addr: 1.0.0.1@853
forward-addr: 68.105.28.12@853
forward-addr: 68.105.29.11@853
forward-addr: 8.8.8.8@853

# Authority zones
# The data for these zones is kept locally, from a file or downloaded.
# The data can be served to downstream clients, or used instead of the
# upstream (which saves a lookup to the upstream).  The first example
# has a copy of the root for local usage.  The second serves example.org
# authoritatively.  zonefile: reads from file (and writes to it if you also
# download it), primary: fetches with AXFR and IXFR, or url to zonefile.
# With allow-notify: you can give additional (apart from primaries and urls)
# sources of notifies.
# auth-zone:
#	name: "."
#	primary: 199.9.14.201         # b.root-servers.net
#	primary: 192.33.4.12          # c.root-servers.net
#	primary: 199.7.91.13          # d.root-servers.net
#	primary: 192.5.5.241          # f.root-servers.net
#	primary: 192.112.36.4         # g.root-servers.net
#	primary: 193.0.14.129         # k.root-servers.net
#	primary: 192.0.47.132         # xfr.cjr.dns.icann.org
#	primary: 192.0.32.132         # xfr.lax.dns.icann.org
#	primary: 2001:500:200::b      # b.root-servers.net
#	primary: 2001:500:2::c        # c.root-servers.net
#	primary: 2001:500:2d::d       # d.root-servers.net
#	primary: 2001:500:2f::f       # f.root-servers.net
#	primary: 2001:500:12::d0d     # g.root-servers.net
#	primary: 2001:7fd::1          # k.root-servers.net
#	primary: 2620:0:2830:202::132 # xfr.cjr.dns.icann.org
#	primary: 2620:0:2d0:202::132  # xfr.lax.dns.icann.org
#	fallback-enabled: yes
#	for-downstream: no
#	for-upstream: yes
# auth-zone:
#	name: "example.org"
#	for-downstream: yes
#	for-upstream: yes
#	zonemd-check: no
#	zonemd-reject-absence: no
#	zonefile: "example.org.zone"

# Views
# Create named views. Name must be unique. Map views to requests using
# the access-control-view option. Views can contain zero or more local-zone
# and local-data options. Options from matching views will override global
# options. Global options will be used if no matching view is found.
# With view-first yes, it will try to answer using the global local-zone and
# local-data elements if there is no view specific match.
# view:
#	name: "viewname"
#	local-zone: "example.com" redirect
#	local-data: "example.com A 192.0.2.3"
#	local-data-ptr: "192.0.2.3 www.example.com"
#	view-first: no
# view:
#	name: "anotherview"
#	local-zone: "example.com" refuse

# DNSCrypt
# To enable, use --enable-dnscrypt to configure before compiling.
# Caveats:
# 1. the keys/certs cannot be produced by Unbound. You can use dnscrypt-wrapper
#   for this: https://github.com/cofyc/dnscrypt-wrapper/blob/master/README.md#usage
# 2. dnscrypt channel attaches to an interface. you MUST set interfaces to
#   listen on `dnscrypt-port` with the follo0wing snippet:
# server:
#     interface: 0.0.0.0@443
#     interface: ::0@443
#
# Finally, `dnscrypt` config has its own section.
# dnscrypt:
#     dnscrypt-enable: yes
#     dnscrypt-port: 443
#     dnscrypt-provider: 2.dnscrypt-cert.example.com.
#     dnscrypt-secret-key: /path/unbound-conf/keys1/1.key
#     dnscrypt-secret-key: /path/unbound-conf/keys2/1.key
#     dnscrypt-provider-cert: /path/unbound-conf/keys1/1.cert
#     dnscrypt-provider-cert: /path/unbound-conf/keys2/1.cert

# CacheDB
# External backend DB as auxiliary cache.
# To enable, use --enable-cachedb to configure before compiling.
# Specify the backend name
# (default is "testframe", which has no use other than for debugging and
# testing) and backend-specific options.  The 'cachedb' module must be
# included in module-config, just before the iterator module.
# cachedb:
#     backend: "testframe"
#     # secret seed string to calculate hashed keys
#     secret-seed: "default"
#
#     # For "redis" backend:
#     # (to enable, use --with-libhiredis to configure before compiling)
#     # redis server's IP address or host name
#     redis-server-host: 127.0.0.1
#     # redis server's TCP port
#     redis-server-port: 6379
#     # timeout (in ms) for communication with the redis server
#     redis-timeout: 100
#     # set timeout on redis records based on DNS response TTL
#     redis-expire-records: no

# IPSet
# Add specify domain into set via ipset.
# To enable:
# o use --enable-ipset to configure before compiling;
# o Unbound then needs to run as root user.
# ipset:
#     # set name for ip v4 addresses
#     name-v4: "list-v4"
#     # set name for ip v6 addresses
#     name-v6: "list-v6"
#

# Dnstap logging support, if compiled in by using --enable-dnstap to configure.
# To enable, set the dnstap-enable to yes and also some of
# dnstap-log-..-messages to yes.  And select an upstream log destination, by
# socket path, TCP or TLS destination.
# dnstap:
# 	dnstap-enable: no
# 	# if set to yes frame streams will be used in bidirectional mode
# 	dnstap-bidirectional: yes
# 	dnstap-socket-path: ""
# 	# if "" use the unix socket in dnstap-socket-path, otherwise,
# 	# set it to "IPaddress[@port]" of the destination.
# 	dnstap-ip: ""
# 	# if set to yes if you want to use TLS to dnstap-ip, no for TCP.
# 	dnstap-tls: yes
# 	# name for authenticating the upstream server. or "" disabled.
# 	dnstap-tls-server-name: ""
# 	# if "", it uses the cert bundle from the main Unbound config.
# 	dnstap-tls-cert-bundle: ""
# 	# key file for client authentication, or "" disabled.
# 	dnstap-tls-client-key-file: ""
# 	# cert file for client authentication, or "" disabled.
# 	dnstap-tls-client-cert-file: ""
# 	dnstap-send-identity: no
# 	dnstap-send-version: no
# 	# if "" it uses the hostname.
# 	dnstap-identity: ""
# 	# if "" it uses the package version.
# 	dnstap-version: ""
# 	dnstap-log-resolver-query-messages: no
# 	dnstap-log-resolver-response-messages: no
# 	dnstap-log-client-query-messages: no
# 	dnstap-log-client-response-messages: no
# 	dnstap-log-forwarder-query-messages: no
# 	dnstap-log-forwarder-response-messages: no

# Response Policy Zones
# RPZ policies. Applied in order of configuration. QNAME, Response IP
# Address, nsdname, nsip and clientip triggers are supported. Supported
# actions are: NXDOMAIN, NODATA, PASSTHRU, DROP, Local Data, tcp-only
# and drop.  Policies can be loaded from a file, or using zone
# transfer, or using HTTP. The respip module needs to be added
# to the module-config, e.g.: module-config: "respip validator iterator".
# rpz:
#     name: "rpz.example.com"
#     zonefile: "rpz.example.com"
#     primary: 192.0.2.0
#     allow-notify: 192.0.2.0/32
#     url: http://www.example.com/rpz.example.org.zone
#     rpz-action-override: cname
#     rpz-cname-override: www.example.org
#     rpz-log: yes
#     rpz-log-name: "example policy"
#     rpz-signal-nxdomain-ra: no
#     for-downstream: no
#     tags: "example"
```

The "#" sign in the black text in the script above is turning off port 53 in the /usr/local/etc/unbound/unbound.conf file.

After we have turned off port 53, now let's continue by editing the named.conf file which is located in the /usr/local/etc/namedb folder.

```
root@router2:~ # ee /usr/local/etc/namedb/named.conf
```

In the original file named.conf we add the following scripts.

```
acl "client_LAN" { 192.168.9.0/24; 127.0.0.1; };
acl IP_LAN { 127.0.0.1; };
auth-nxdomain no;
dnssec-validation yes;
recursion yes;
allow-recursion { client_LAN; };
allow-query { client_LAN; };
allow-query-cache { client_LAN; };
allow-transfer { none; };
listen-on port 53 { IP_LAN; };
logging {
     channel default_log {
          file "/var/named/log/default" versions 3 size 20m;
          print-time yes;
          print-category yes;
          print-severity yes;
          severity info;
     };
     channel auth_servers_log {
          file "/var/named/log/auth_servers" versions 100 size 20m;
          print-time yes;
          print-category yes;
          print-severity yes;
          severity info;
     };
     channel dnssec_log {
          file "/var/named/log/dnssec" versions 3 size 20m;
          print-time yes;
          print-category yes;
          print-severity yes;
          severity info;
     };
     channel zone_transfers_log {
          file "/var/named/log/zone_transfers" versions 3 size 20m;
          print-time yes;
          print-category yes;
          print-severity yes;
          severity info;
     };
     channel ddns_log {
          file "/var/named/log/ddns" versions 3 size 20m;
          print-time yes;
          print-category yes;
          print-severity yes;
          severity info;
     };
     channel client_security_log {
          file "/var/named/log/client_security" versions 3 size 20m;
          print-time yes;
          print-category yes;
          print-severity yes;
          severity info;
     };
     channel rate_limiting_log {
          file "/var/named/log/rate_limiting" versions 3 size 20m;
          print-time yes;
          print-category yes;
          print-severity yes;
          severity info;
     };
     channel rpz_log {
          file "/var/named/log/rpz" versions 3 size 20m;
          print-time yes;
          print-category yes;
          print-severity yes;
          severity info;
     };
     channel dnstap_log {
          file "/var/named/log/dnstap" versions 3 size 20m;
          print-time yes;
          print-category yes;
          print-severity yes;
          severity info;
     };

     channel queries_log {
          file "/var/named/log/queries" versions 600 size 20m;
          print-time yes;
          print-category yes;
          print-severity yes;
          severity info;
     };
     channel query-errors_log {
          file "/var/named/log/query-errors" versions 5 size 20m;
          print-time yes;
          print-category yes;
          print-severity yes;
          severity dynamic;
     };
     channel default_syslog {
          print-time yes;
          print-category yes;
          print-severity yes;
          syslog daemon;
          severity info;
     };
     channel default_debug {
          print-time yes;
          print-category yes;
          print-severity yes;
          file "named.run";
          severity dynamic;
     };
     category default { default_syslog; default_debug; default_log; };
     category config { default_syslog; default_debug; default_log; };
     category dispatch { default_syslog; default_debug; default_log; };
     category network { default_syslog; default_debug; default_log; };
     category general { default_syslog; default_debug; default_log; };
     category zoneload { default_syslog; default_debug; default_log; };
     category resolver { auth_servers_log; default_debug; };       
     category cname { auth_servers_log; default_debug; };       
     category delegation-only { auth_servers_log; default_debug; };
     category lame-servers { auth_servers_log; default_debug; };
     category edns-disabled { auth_servers_log; default_debug; };
     category dnssec { dnssec_log; default_debug; };
     category notify { zone_transfers_log; default_debug; };       
     category xfer-in { zone_transfers_log; default_debug; };       
     category xfer-out { zone_transfers_log; default_debug; };
     category update{ ddns_log; default_debug; };
     category update-security { ddns_log; default_debug; };
     category client{ client_security_log; default_debug; };       
     category security { client_security_log; default_debug; };
     category rate-limit { rate_limiting_log; default_debug; };       
     category spill { rate_limiting_log; default_debug; };       
     category database { rate_limiting_log; default_debug; };
     category rpz { rpz_log; default_debug; };
     category dnstap { dnstap_log; default_debug; };
     category trust-anchor-telemetry { default_syslog; default_debug; default_log; };
     category queries { queries_log; };
     category query-errors {query-errors_log; };
};
```

We restart and test the bind program.

```
root@router2:~ # service named restart
root@router2:~ # dig google.com
```

After we tested, it turned out that the bind program was not running, because the bind IP 192.168.5.2 and port 53 could not answer google.com's DNS.
<br><br/>
## C. Bind Forwarding
So that the Bind DNS server can RUNNING, we have to create a forward script. In this tutorial we will forward the Bind program to the Unbound server with IP 192.168.5.2 port 853.

To forward we have to edit the named.conf file.

```
root@router2:~ # ee /usr/local/etc/namedb/named.conf
```

After the unbound conf file is open, add the following forward script on the file /usr/local/etc/namedb/named.conf

```
zone "." {
  type forward;
  forward first;
  forwarders {
    192.168.5.2 port 853;    
  };
};
```

We restart and test the bind program.

```
root@router2:~ # service named restart
root@router2:~ # dig google.com
```

The answer to google.com's DNS is the BIND Program with IP 192.168.5.2 Port 53
Now we test the unbound program running on port 853 IP 192.168.5.2.

```
root@router2:~ # service unbound restart
root@router2:~ # dig -p 853 google.com @192.168.5.2

; <<>> DiG 9.18.19 <<>> -p 853 google.com @192.168.5.2
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2444
;; flags: qr rd ra; QUERY: 1, ANSWER: 6, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             20      IN      A       142.250.4.113
google.com.             20      IN      A       142.250.4.100
google.com.             20      IN      A       142.250.4.139
google.com.             20      IN      A       142.250.4.102
google.com.             20      IN      A       142.250.4.101
google.com.             20      IN      A       142.250.4.138

;; Query time: 0 msec
;; SERVER: 192.168.5.2#853(192.168.5.2) (UDP)
;; WHEN: Fri Oct 27 20:15:45 WIB 2023
;; MSG SIZE  rcvd: 135
```

Pay attention to the script above in red, the answer to the google.com DNS is the unbound program with IP 192.168.5.2 Port 853.

All programs are running, both the BIND and UNBOUND programs, everything is running normally. This means that we have succeeded in making the Unbound program the BACKEND of the BIND program, and making the BIND program the FRONTEND for the CLIENT.

Now our PC, laptop or cellphone can use DNS from BIND, namely with IP 192.168.5.2 port 53.

Below I will provide the COMPLETE SCRIPT of the named.conf file.

```
// Refer to the named.conf(5) and named(8) man pages, and the documentation
// in /usr/local/share/doc/bind for more details.
//
// If you are going to set up an authoritative server, make sure you
// understand the hairy details of how DNS works.  Even with
// simple mistakes, you can break connectivity for affected parties,
// or cause huge amounts of useless Internet traffic.

acl "client_LAN" { 192.168.5.0/24; 127.0.0.1; };
acl IP_LAN { 192.168.5.2; };


options {
	// All file and path names are relative to the chroot directory,
	// if any, and should be fully qualified.
	directory	"/usr/local/etc/namedb/working";
	pid-file	"/var/run/named/pid";
	dump-file	"/var/dump/named_dump.db";
	statistics-file	"/var/stats/named.stats";
	auth-nxdomain no;
	dnssec-validation yes;
	recursion yes;
	allow-recursion { client_LAN; };
	allow-query { client_LAN; };
	allow-query-cache { client_LAN; };
        allow-transfer { none; };

// If named is being used only as a local resolver, this is a safe default.
// For named to be accessible to the network, comment this option, specify
// the proper IP address, or delete this option.
//	listen-on	{ 127.0.0.1; };
	listen-on port 53 { IP_LAN; };

// If you have IPv6 enabled on this system, uncomment this option for
// use as a local resolver.  To give access to the network, specify
// an IPv6 address, or the keyword "any".
//	listen-on-v6	{ ::1; };

// These zones are already covered by the empty zones listed below.
// If you remove the related empty zones below, comment these lines out.
	disable-empty-zone "255.255.255.255.IN-ADDR.ARPA";
	disable-empty-zone "0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.IP6.ARPA";
	disable-empty-zone "1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.IP6.ARPA";

// If you've got a DNS server around at your upstream provider, enter
// its IP address here, and enable the line below.  This will make you
// benefit from its cache, thus reduce overall DNS traffic in the Internet.
/*
	forwarders {
		127.0.0.1;
	};
*/

// If the 'forwarders' clause is not empty the default is to 'forward first'
// which will fall back to sending a query from your local server if the name
// servers in 'forwarders' do not have the answer.  Alternatively you can
// force your name server to never initiate queries of its own by enabling the
// following line:
//	forward only;

// If you wish to have forwarding configured automatically based on
// the entries in /etc/resolv.conf, uncomment the following line and
// set named_auto_forward=yes in /etc/rc.conf.  You can also enable
// named_auto_forward_only (the effect of which is described above).
//	include "/usr/local/etc/namedb/auto_forward.conf";

	/*
	   Modern versions of BIND use a random UDP port for each outgoing
	   query by default in order to dramatically reduce the possibility
	   of cache poisoning.  All users are strongly encouraged to utilize
	   this feature, and to configure their firewalls to accommodate it.

	   AS A LAST RESORT in order to get around a restrictive firewall
	   policy you can try enabling the option below.  Use of this option
	   will significantly reduce your ability to withstand cache poisoning
	   attacks, and should be avoided if at all possible.

	   Replace NNNNN in the example with a number between 49160 and 65530.
	*/
	// query-source address * port NNNNN;
};

// If you enable a local name server, don't forget to enter 127.0.0.1
// first in your /etc/resolv.conf so this server will be queried.
// Also, make sure to enable it in /etc/rc.conf.

// The traditional root hints mechanism. Use this, OR the secondary zones below.
zone "." { type hint; file "/usr/local/etc/namedb/named.root"; };

/*	Slaving the following zones from the root name servers has some
	significant advantages:
	1. Faster local resolution for your users
	2. No spurious traffic will be sent from your network to the roots
	3. Greater resilience to any potential root server failure/DDoS

	On the other hand, this method requires more monitoring than the
	hints file to be sure that an unexpected failure mode has not
	incapacitated your server.  Name servers that are serving a lot
	of clients will benefit more from this approach than individual
	hosts.  Use with caution.

	To use this mechanism, uncomment the entries below, and comment
	the hint zone above.

	As documented at http://dns.icann.org/services/axfr/ these zones:
	"." (the root), ARPA, IN-ADDR.ARPA, IP6.ARPA, and a few others
	are available for AXFR from these servers on IPv4 and IPv6:
	xfr.lax.dns.icann.org, xfr.cjr.dns.icann.org
*/
/*
zone "." {
	type secondary;
	file "/usr/local/etc/namedb/secondary/root.secondary";
	primaries {
		192.0.32.132;           // lax.xfr.dns.icann.org
		2620:0:2d0:202::132;    // lax.xfr.dns.icann.org
		192.0.47.132;           // iad.xfr.dns.icann.org
		2620:0:2830:202::132;   // iad.xfr.dns.icann.org
	};
	notify no;
};
zone "arpa" {
	type secondary;
	file "/usr/local/etc/namedb/secondary/arpa.secondary";
	primaries {
		192.0.32.132;           // lax.xfr.dns.icann.org
		2620:0:2d0:202::132;    // lax.xfr.dns.icann.org
		192.0.47.132;           // iad.xfr.dns.icann.org
		2620:0:2830:202::132;   // iad.xfr.dns.icann.org
	};
	notify no;
};
zone "in-addr.arpa" {
	type secondary;
	file "/usr/local/etc/namedb/secondary/in-addr.arpa.secondary";
	primaries {
		192.0.32.132;           // lax.xfr.dns.icann.org
		2620:0:2d0:202::132;    // lax.xfr.dns.icann.org
		192.0.47.132;           // iad.xfr.dns.icann.org
		2620:0:2830:202::132;   // iad.xfr.dns.icann.org
	};
	notify no;
};
zone "ip6.arpa" {
	type secondary;
	file "/usr/local/etc/namedb/secondary/ip6.arpa.secondary";
	primaries {
		192.0.32.132;           // lax.xfr.dns.icann.org
		2620:0:2d0:202::132;    // lax.xfr.dns.icann.org
		192.0.47.132;           // iad.xfr.dns.icann.org
		2620:0:2830:202::132;   // iad.xfr.dns.icann.org
	};
	notify no;
};
*/

/*	Serving the following zones locally will prevent any queries
	for these zones leaving your network and going to the root
	name servers.  This has two significant advantages:
	1. Faster local resolution for your users
	2. No spurious traffic will be sent from your network to the roots
*/
// RFCs 1912, 5735 and 6303 (and BCP 32 for localhost)
zone "localhost"	{ type primary; file "/usr/local/etc/namedb/primary/localhost-forward.db"; };
zone "127.in-addr.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/localhost-reverse.db"; };
zone "255.in-addr.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };

// RFC 1912-style zone for IPv6 localhost address (RFC 6303)
zone "0.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/localhost-reverse.db"; };

// "This" Network (RFCs 1912, 5735 and 6303)
zone "0.in-addr.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };

// Private Use Networks (RFCs 1918, 5735 and 6303)
zone "10.in-addr.arpa"	   { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "16.172.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "17.172.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "18.172.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "19.172.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "20.172.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "21.172.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "22.172.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "23.172.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "24.172.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "25.172.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "26.172.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "27.172.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "28.172.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "29.172.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "30.172.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "31.172.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "168.192.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };

// Shared Address Space (RFC 6598)
zone "64.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "65.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "66.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "67.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "68.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "69.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "70.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "71.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "72.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "73.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "74.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "75.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "76.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "77.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "78.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "79.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "80.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "81.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "82.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "83.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "84.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "85.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "86.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "87.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "88.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "89.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "90.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "91.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "92.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "93.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "94.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "95.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "96.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "97.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "98.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "99.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "100.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "101.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "102.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "103.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "104.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "105.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "106.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "107.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "108.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "109.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "110.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "111.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "112.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "113.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "114.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "115.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "116.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "117.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "118.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "119.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "120.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "121.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "122.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "123.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "124.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "125.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "126.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "127.100.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };

// Link-local/APIPA (RFCs 3927, 5735 and 6303)
zone "254.169.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };

// IETF protocol assignments (RFCs 5735 and 5736)
zone "0.0.192.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };

// TEST-NET-[1-3] for Documentation (RFCs 5735, 5737 and 6303)
zone "2.0.192.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "100.51.198.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "113.0.203.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };

// IPv6 Example Range for Documentation (RFCs 3849 and 6303)
zone "8.b.d.0.1.0.0.2.ip6.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };

// Router Benchmark Testing (RFCs 2544 and 5735)
zone "18.198.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "19.198.in-addr.arpa" { type primary; file "/usr/local/etc/namedb/primary/empty.db"; };

// IANA Reserved - Old Class E Space (RFC 5735)
zone "240.in-addr.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "241.in-addr.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "242.in-addr.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "243.in-addr.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "244.in-addr.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "245.in-addr.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "246.in-addr.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "247.in-addr.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "248.in-addr.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "249.in-addr.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "250.in-addr.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "251.in-addr.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "252.in-addr.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "253.in-addr.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "254.in-addr.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };

// IPv6 Unassigned Addresses (RFC 4291)
zone "1.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "3.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "4.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "5.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "6.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "7.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "8.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "9.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "a.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "b.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "c.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "d.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "e.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "0.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "1.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "2.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "3.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "4.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "5.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "6.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "7.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "8.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "9.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "a.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "b.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "0.e.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "1.e.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "2.e.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "3.e.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "4.e.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "5.e.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "6.e.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "7.e.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };

// IPv6 ULA (RFCs 4193 and 6303)
zone "c.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "d.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };

// IPv6 Link Local (RFCs 4291 and 6303)
zone "8.e.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "9.e.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "a.e.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "b.e.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };

// IPv6 Deprecated Site-Local Addresses (RFCs 3879 and 6303)
zone "c.e.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "d.e.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "e.e.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };
zone "f.e.f.ip6.arpa"	{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };

// IP6.INT is Deprecated (RFC 4159)
zone "ip6.int"		{ type primary; file "/usr/local/etc/namedb/primary/empty.db"; };

// NB: Do not use the IP addresses below, they are faked, and only
// serve demonstration/documentation purposes!
//
// Example secondary zone config entries.  It can be convenient to become
// a secondary at least for the zone your own domain is in.  Ask
// your network administrator for the IP address of the responsible
// primary name server.
//
// Do not forget to include the reverse lookup zone!
// This is named after the first bytes of the IP address, in reverse
// order, with ".IN-ADDR.ARPA" appended, or ".IP6.ARPA" for IPv6.
//
// Before starting to set up a primary zone, make sure you fully
// understand how DNS and BIND work.  There are sometimes
// non-obvious pitfalls.  Setting up a secondary zone is usually simpler.
//
// NB: Don't blindly enable the examples below. :-)  Use actual names
// and addresses instead.

/* An example dynamic zone
key "exampleorgkey" {
	algorithm hmac-md5;
	secret "sf87HJqjkqh8ac87a02lla==";
};
zone "example.org" {
	type primary;
	allow-update {
		key "exampleorgkey";
	};
	file "/usr/local/etc/namedb/dynamic/example.org";
};
*/

/* Example of a secondary reverse zone
zone "1.168.192.in-addr.arpa" {
	type secondary;
	file "/usr/local/etc/namedb/secondary/1.168.192.in-addr.arpa";
	primaries {
		192.168.1.1;
	};
};
*/

logging {
     channel default_log {
          file "/var/named/log/default" versions 3 size 20m;
          print-time yes;
          print-category yes;
          print-severity yes;
          severity info;
     };
     channel auth_servers_log {
          file "/var/named/log/auth_servers" versions 100 size 20m;
          print-time yes;
          print-category yes;
          print-severity yes;
          severity info;
     };
     channel dnssec_log {
          file "/var/named/log/dnssec" versions 3 size 20m;
          print-time yes;
          print-category yes;
          print-severity yes;
          severity info;
     };
     channel zone_transfers_log {
          file "/var/named/log/zone_transfers" versions 3 size 20m;
          print-time yes;
          print-category yes;
          print-severity yes;
          severity info;
     };
     channel ddns_log {
          file "/var/named/log/ddns" versions 3 size 20m;
          print-time yes;
          print-category yes;
          print-severity yes;
          severity info;
     };
     channel client_security_log {
          file "/var/named/log/client_security" versions 3 size 20m;
          print-time yes;
          print-category yes;
          print-severity yes;
          severity info;
     };
     channel rate_limiting_log {
          file "/var/named/log/rate_limiting" versions 3 size 20m;
          print-time yes;
          print-category yes;
          print-severity yes;
          severity info;
     };
     channel rpz_log {
          file "/var/named/log/rpz" versions 3 size 20m;
          print-time yes;
          print-category yes;
          print-severity yes;
          severity info;
     };
     channel dnstap_log {
          file "/var/named/log/dnstap" versions 3 size 20m;
          print-time yes;
          print-category yes;
          print-severity yes;
          severity info;
     };

     channel queries_log {
          file "/var/named/log/queries" versions 600 size 20m;
          print-time yes;
          print-category yes;
          print-severity yes;
          severity info;
     };
     channel query-errors_log {
          file "/var/named/log/query-errors" versions 5 size 20m;
          print-time yes;
          print-category yes;
          print-severity yes;
          severity dynamic;
     };
     channel default_syslog {
          print-time yes;
          print-category yes;
          print-severity yes;
          syslog daemon;
          severity info;
     };
     channel default_debug {
          print-time yes;
          print-category yes;
          print-severity yes;
          file "named.run";
          severity dynamic;
     };
     category default { default_syslog; default_debug; default_log; };
     category config { default_syslog; default_debug; default_log; };
     category dispatch { default_syslog; default_debug; default_log; };
     category network { default_syslog; default_debug; default_log; };
     category general { default_syslog; default_debug; default_log; };
     category zoneload { default_syslog; default_debug; default_log; };
     category resolver { auth_servers_log; default_debug; };       
     category cname { auth_servers_log; default_debug; };       
     category delegation-only { auth_servers_log; default_debug; };
     category lame-servers { auth_servers_log; default_debug; };
     category edns-disabled { auth_servers_log; default_debug; };
     category dnssec { dnssec_log; default_debug; };
     category notify { zone_transfers_log; default_debug; };       
     category xfer-in { zone_transfers_log; default_debug; };       
     category xfer-out { zone_transfers_log; default_debug; };
     category update{ ddns_log; default_debug; };
     category update-security { ddns_log; default_debug; };
     category client{ client_security_log; default_debug; };       
     category security { client_security_log; default_debug; };
     category rate-limit { rate_limiting_log; default_debug; };       
     category spill { rate_limiting_log; default_debug; };       
     category database { rate_limiting_log; default_debug; };
     category rpz { rpz_log; default_debug; };
     category dnstap { dnstap_log; default_debug; };
     category trust-anchor-telemetry { default_syslog; default_debug; default_log; };
     category queries { queries_log; };
     category query-errors {query-errors_log; };
};

zone "." {
  type forward;
  forward first;
  forwarders {
    192.168.5.2 port 853;    
  };
};
```
