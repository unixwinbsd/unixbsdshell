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

### e. Create Users
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
<br><br/>
## 2. Create user Freeradius with MySQL server
When FreeRadius is used together with Mariadb or MySQL, Freeradius will use a database which is usually called 'radius' and within that database there is a database table called 'radcheck'. This table is the table we need to use to interact between Mariadb and Freeradius, because it contains all the user accounts that can be authenticated with FreeRadius.

It's important to remember that like many other things, you can choose the username to use, the database name for something, and you can even choose to use a remote MySQL server! However for this tutorial we will assume that MySQL and FreeRadius are on the same server, and the database is called 'radius' and the user account we will use with MySQL is root.

To create a radius database, first log in to the Mariadb database with the root account, after that create a radius database, see the example below.

```
ns3# mysql -u root -p
MariaDB [(none)]> CREATE DATABASE radius;
MariaDB [(none)]> CREATE USER 'userradius'@'localhost' IDENTIFIED BY 'radius123';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON radius.* TO 'userradius'@'localhost';
MariaDB [(none)]> FLUSH PRIVILEGES;
MariaDB [(none)]> exit;
```

### a. Create Freeradius schema
For the first step, we need to create a schema.sql file, which contains the tables that Freeradius needs to communicate with Maridb. You can find an example of a schema.sql file at /etc/raddb/mods-config/sql/main/mysql/schema.sql.

Import the schema.sql script into the newly created radius database.

```
ns3# mysql -uroot -prouter123 radius < /etc/raddb/mods-config/sql/main/mysql/schema.sql
```

root: Mariadb server user
router123: Mariadb server password

### b. Create db user and client
Now we will create a new user which will be stored in the radius database. Run the insert command to add a new user. For this example the user we will add is Beyoncé, and she will have the following login details:

Username: testuser
Password: mariadb123

```
ns3# mysql -u root -prouter123
MariaDB [(none)]> use radius;
MariaDB [radius]> INSERT INTO radcheck (id, username, attribute, op, value) VALUES (1001,'testuser','Cleartext-Password',':=','mariadb123');
```

```
MariaDB [radius]> INSERT INTO nas (nasname, shortname, type, ports, secret) VALUES ('192.168.5.3', 'router', 'other', 1812,'router123');
```

### c. Create Freeradius SQL Module
One of the functions of the Radius module is to connect Freeradius with other applications such as the Mariadb database. In this example we will connect freeradius with Mariadb. To make this connection, you change the /etc/raddb/mods-available/sql file. You can see an example of the complete script below.

```
sql {
	dialect = "mysql"
	driver = "rlm_sql_mysql"
	sqlite {
		filename = "/tmp/freeradius.db"
		busy_timeout = 200
		bootstrap = "${modconfdir}/${..:name}/main/sqlite/schema.sql"
	}

	mysql {
		warnings = auto
	}

	server = "localhost"
	port = 3306
	login = "userradius"
	password = "radius123"
	radius_db = "radius"
	acct_table1 = "radacct"
	acct_table2 = "radacct"
	postauth_table = "radpostauth"
	authcheck_table = "radcheck"
	groupcheck_table = "radgroupcheck"
	authreply_table = "radreply"
	groupreply_table = "radgroupreply"
	usergroup_table = "radusergroup"
	delete_stale_sessions = yes

	pool {
		start = ${thread[pool].start_servers}
		min = ${thread[pool].min_spare_servers}
		max = ${thread[pool].max_servers}
		spare = ${thread[pool].max_spare_servers}
		uses = 0
		retry_delay = 30
		lifetime = 0
		idle_timeout = 60
		max_retries = 5
	}
	read_clients = yes
	client_table = "nas"
	group_attribute = "SQL-Group"
	$INCLUDE ${modconfdir}/${.:name}/main/${dialect}/queries.conf
}
```

Then you continue by creating a symlink.

```
ns3# ln -s /etc/raddb/mods-available/sql /etc/raddb/mods-enabled/
```

### d. Enable sql option
Because in this article we will connect Freeradius to the Mariadb database server, we have to activate the "sql" option in the "default" and "inner-tunnel" files. The file is located in the /etc/raddb/sites-available directory.

In the "default" and "inner-tunnel" file scripts, you remove the “#” and “-” signs (“#sql” and “-sql”) so that it just becomes a “sql” script.

### e. Test Freeradius SQL Module
This section is the most important step, because we will test whether the Freeradius server is connected to the Mariadb server. We will test it with the user and password that we created with the SQL command above. Run the following command to test the Freeradius server.

```
ns3# radtest testuser mariadb123 192.168.5.3 1812 router123
```

In conclusion, Freeradius is a powerful and flexible tool for improving network security and performance. Freeradius allows unique credentials for each user, as there is no unified password that is shared among a number of people, this can prevent the threat of hackers infiltrating your network.
