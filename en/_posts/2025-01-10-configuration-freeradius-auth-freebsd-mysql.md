---
title: Configuring OpenBSD to Use Freeradius Auth with MySQL Server
date: "2025-01-10 13:11:10 +0100"
id: configuration-freeradius-auth-freebsd-mysql
lang: en
layout: single
categories:
  - OpenBSD
tags: "openbsd, freebsd, mysql server, freeradius"
excerpt: FreeRADIUS is an excellent free server and provides centralized authentication and authorization services for devices including switches, routers, VPN gateways, and Wi-Fi access points
keywords: freebsd, openbsd, freeradius, auth, mysql server, mysql, database, radius
---


In modern enterprise network architecture, it has a complex and complicated structure. Lots of devices are connected to each other simultaneously, making it convenient for offenders and anyone with illegal intentions. To overcome this problem, a mechanism for recording subject information in the network was developed. A RADIUS server that can provide 3 security functions: authentication, authorization and accounting. Every step in the system will be logged and all records and entry points will be monitored.

FreeRADIUS is an excellent free server and provides centralized authentication and authorization services for devices including switches, routers, VPN gateways, and Wi-Fi access points. Its unique ability to manage user access to network resources based on various parameters such as identity, location, device characteristics and time of day, makes it a versatile solution for enterprise, education and service provider networks.

The server supports multiple authentication methods and can integrate with external databases such as SQL, LDAP, and Kerberos to efficiently store and retrieve user and device information. With its scalability, flexibility and reliability, FreeRADIUS remains the top choice for organizations requiring a reliable and customizable RADIUS solution.
<br><br/>
## 1. Installing and configuring FreeRADIUS
### a. Update package PKG
Before installing any software, we need to update the indexes and packages in OpenBSD.

```
ns3# pkg_add -uvi
```

### b. Install Freeradius
Install the Freeradius package with the pkg_add command.

```
ns3# pkg_add freeradius debug-freeradius-mysql freeradius-mysql
```

### c. Edit /etc/raddb/radiusd.conf
Freeradius' main configuration file is "radiusd.conf". Before you configure other files, you must first edit radiusd.conf.

Below is an example of the complete radiusd.conf script.

```
prefix = /usr/local
exec_prefix = ${prefix}
sysconfdir = /etc
localstatedir = /var
sbindir = ${exec_prefix}/sbin
logdir = ${localstatedir}/log/radius
raddbdir = ${sysconfdir}/raddb
radacctdir = ${logdir}/radacct

name = radiusd
confdir = ${raddbdir}
modconfdir = ${confdir}/mods-config
certdir = ${confdir}/certs
cadir   = ${confdir}/certs
run_dir = ${localstatedir}/run/${name}
db_dir = ${raddbdir}

#libdir = /usr/local/lib/freeradius/freeradius
pidfile = ${run_dir}/${name}.pid
max_request_time = 30

cleanup_delay = 5
max_requests = 16384
hostname_lookups = no

log {
	destination = files
	colourise = yes
	file = ${logdir}/radius.log
	syslog_facility = daemon
	stripped_names = no
	auth = no
	auth_badpass = no
	auth_goodpass = no
	msg_denied = "You are already logged in - access denied"
}
checkrad = ${sbindir}/checkrad

ENV {
}

security {
###	chroot = /etc/raddb
	user = _freeradius
	group = _freeradius
	allow_core_dumps = no
	max_attributes = 200
	reject_delay = 1
	status_server = yes
	allow_vulnerable_openssl = no
}

proxy_requests  = yes
$INCLUDE proxy.conf

$INCLUDE clients.conf

thread pool {
	start_servers = 5
	max_servers = 32
	min_spare_servers = 3
	max_spare_servers = 10
	max_requests_per_server = 0
	auto_limit_acct = no
}

#$INCLUDE trigger.conf
modules {
#	$INCLUDE mods-enabled/sql
	$INCLUDE mods-enabled/
}

instantiate {
#	daily
}

policy {
	$INCLUDE policy.d/
}

$INCLUDE sites-enabled/
```

### d. Create Client
As a basic example, we will create a client that can connect to the Freeradius server. In this example, we will create two clients that can connect directly to Freeradius:
localhost client (127.0.0.1)
pfsense client (192.168.5.3).

Below is an example of a complete /etc/raddb/clients.conf script.

```
client localhost {
	ipaddr = 127.0.0.1
#	ipv6addr = ::	# any.  ::1 == localhost
	proto = *
	secret = testing123
	require_message_authenticator = no
#	shortname = localhost
	nas_type	 = other	# localhost isn't usually a NAS...
	limit {
		max_connections = 16
		lifetime = 0
		idle_timeout = 30
	}
}

client pfsense {
	ipaddr = 192.168.5.3
	secret = router123
	shortname = router
	limit {
		max_connections = 16
		lifetime = 0
		idle_timeout = 30
	}
}
```

### 
e. Create Users
Next, create a user and password that can use Freeradius. Below is the complete script /etc/raddb/mods-config/files/authorize.

```
#bob	Cleartext-Password := "hello"
#	Reply-Message := "Hello, %{User-Name}"
#"John Doe"	Cleartext-Password := "hello"
#		Reply-Message = "Hello, %{User-Name}"

steve Cleartext-Password := "steve123"
"MaryRose" Cleartext-Password := "mary123"
```

### f. Ownership
By default the OpenBSD system has created the user and group _freeradius:_freeradius. Run the command below to grant ownership rights to Freeradius.

```
ns3# chown -R _freeradius:_freeradius /etc/raddb/
ns3# chown -R _freeradius:_freeradius /var/log/radius/
```

### g. Activate Freeradius
Then we run Freeradius so that it can run as a daemon on OpenBSD. Run the following command to activate Freeradius.

```
ns3# rcctl enable freeradius
ns3# rcctl restart freeradius
```

### h. Test Freeradius
In this section we will examine users who can connect to the Freeradisu server. Look at the example below to check each user connected to Freeradius.

```
ns3# radtest steve steve123 192.168.5.3 1812 router123
Sent Access-Request Id 29 from 0.0.0.0:482e to 192.168.5.3:1812 length 75
	User-Name = "steve"
	User-Password = "steve123"
	NAS-IP-Address = 192.168.5.3
	NAS-Port = 1812
	Message-Authenticator = 0x00
	Cleartext-Password = "steve123"
Received Access-Accept Id 29 from 192.168.5.3:714 to 192.168.5.3:18478 length 20
```

```
ns3# radtest steve steve123 127.0.0.1 1812 testing123
ns3# radtest steve steve123 localhost 1812 testing123
ns3# radtest MaryRose mary123 192.168.5.3 1812 router123
```

## 2. Create user Freeradius with MySQL server

When FreeRadius is used together with Mariadb or MySQL, Freeradius will use a database which is usually called 'radius' and within that database there is a database table called 'radcheck'. This table is the table we need to use to interact between Mariadb and Freeradius, because it contains all the user accounts that can be authenticated with FreeRadius.

It's important to remember that like many other things, you can choose the username to use, the database name for something, and you can even choose to use a remote MySQL server! However for this tutorial we will assume that MySQL and FreeRadius are on the same server, and the database is called 'radius' and the user account we will use with MySQL is root.

To create a radius database, first log in to the Mariadb database with the root account, after that create a radius database, see the example below.



