---
title: Используйте цепочки прокси с Tor для анонимного серфинга в GhostBSD
date: "2022-11-01 17:00:00 +0300"
lang: ru
id: ghostbsd-proxychain-with-tor
excerpt: "ProxyChains — это программа UNIX, которая объединяет сетевые функции libc в динамически связанную программу с помощью предварительно загруженной DLL и перенаправляет соединения через прокси-сервер SOCKS4a/5 или HTTP."
tags: "ghostbsd, freebsd, proxy, proxychain, configuration, setup"
keywords: ghostbsd, freebsd, proxy, proxychain, configuration, setup
---


ProxyChains — это программа UNIX, которая связывает сетевые функции libc в динамически связанную программу через предварительно загруженную DLL и перенаправляет соединение через SOCKS4a/5 или HTTP-прокси. Proxychains поддерживает только протокол TCP, протоколы UDM и ICMP для Proxychains не поддерживаются. Proxychains может работать в любой операционной системе, например, Linux, NetBSD, FreeBSD, OpenBSD, DragonFlyBSD, HardenedBSD или Mac OS

Во время работы [Proxychains](https://www.proxifier.com/docs/win-v4/chain.html) выполняет очень сложный процесс, включающий использование прокси-сервера для проксирования на другой сервер и многократное выполнение этого процесса для создания достаточного количества уровней маскировки, обработки пакетов и запросов, чтобы обеспечить высокий уровень анонимности.

У Proxychains есть два основных варианта использования:
- Создайте анонимных пользователей.
- Заставьте их обходить ограниченные блокировки, которые могут обнаружить и заблокировать обычные прокси.

В этом руководстве мы объясним, как установить, настроить и использовать Proxychains с [бэкэндом Tor](https://www.inchimediatama.org/2025/02/konfigurasi-3proxy-dengan-tor-di-freebsd.html), чтобы всегда гарантировать конфиденциальность, и даже если сервер идентифицирует нашу личность, мы автоматически немедленно изменим личность и продолжим сохранять анонимность.

## А. Технические характеристики системы
- OS: GhostBSD 23.06.01
- IP Server: 192.168.5.2
- Hostname: ns1
- Domain: unixexplore.com
- TOR version: Tor 0.4.7.13
- Proxychains: proxychains-3.1_3

## Б. Установка TOR

В этом руководстве мы будем использовать TOR в качестве бэкэнда Proxychain, поэтому всякий раз, когда клиент обращается к веб-сайту, этот веб-сайт будет обслуживаться Proxychain в качестве фронтэнда, а прокси-соединение будет использовать TOR в качестве бэкэнда.

Чтобы установить TOR на GhostBSD, войдите в систему как пользователь root. На рабочем столе GhostBSD щелкните правой кнопкой мыши и выберите «Открыть в терминале», чтобы войти в командную строку GhostBSD. Когда появится меню командной строки, введите следующий скрипт.


```cs
ns3-ghostbsd-pc@ns3 /u/h/n/Desktop> su root
Password: masukkan password root
root@ns3:/usr/home/ns3-ghostbsd-pc/Desktop #
```

Если появляется слово root, например «root@ns3:/usr/home/ns3-ghostbsd-pc/Desktop #» (см. скрипт выше), это означает, что вы находитесь в системе как пользователь root. После активации в качестве пользователя root установите TOR, как показано в примере ниже.


```cs
root@ns3:/usr/home/ns3-ghostbsd-pc/Desktop # pkg install tor
Updating GhostBSD repository catalogue...
GhostBSD repository is up to date.
All repositories are up to date.
Checking integrity... done (0 conflicting)
The following 1 package(s) will be affected (of 0 checked):

New packages to be INSTALLED:
	tor: 0.4.7.13

Number of packages to be installed: 1

The process will require 14 MiB more space.

Proceed with this action? [y/N]: y
[1/1] Installing tor-0.4.7.13...
===> Creating groups.
Using existing group '_tor'.
===> Creating users
Using existing user '_tor'.
===> Creating homedir(s)
[1/1] Extracting tor-0.4.7.13: 100%
=====
Message from tor-0.4.7.13:

--
To enable the tor server, set tor_enable="YES" in your /etc/rc.conf
and edit /usr/local/etc/tor/torrc as desired. (However, note that the
/usr/local/etc/rc.d/tor rc.subr script can override some torrc
options: see that script for details.) To use the torify script, install
the net/torsocks port.

Tor users are strongly advised to prevent traffic analysis that
exploits sequential IP IDs by setting:

sysctl net.inet.ip.random_id=1

(see sysctl.conf(5)).

In order to run additional, independent instances of tor on the same machine
set tor_instances="inst1 inst2 ..." in your /etc/rc.conf, and create the
corresponding additional configuration files /usr/local/etc/tor/torrc@inst1, ...

Alternatively, you can use the extended instance definition to specify all
instance parameteres explicitly:
inst_name{:inst_conf:inst_user:inst_group:inst_pidfile:inst_data_dir}

Wait until the installation process is complete, then continue by installing torsocks.

root@ns3:/usr/home/ns3-ghostbsd-pc/Desktop # pkg install torsocks
Updating GhostBSD repository catalogue...
GhostBSD repository is up to date.
All repositories are up to date.
The following 1 package(s) will be affected (of 0 checked):

New packages to be INSTALLED:
	torsocks: 2.3.0_1

Number of packages to be installed: 1

71 KiB to be downloaded.

Proceed with this action? [y/N]: y
[1/1] Fetching torsocks-2.3.0_1.pkg: 100%   71 KiB  36.3kB/s    00:02    
Checking integrity... done (0 conflicting)
[1/1] Installing torsocks-2.3.0_1...
[1/1] Extracting torsocks-2.3.0_1: 100%
=====
Message from torsocks-2.3.0_1:

--
You installed torsocks, which is part of the Tor Project.
If you have non-local or non-standard Tor SOCKS server location,
please edit /usr/local/etc/tor/torsocks.conf

To run most programs through Tor launch them like this:
	torsocks [any executable]
For example you can use ssh to a some.host.com by doing:
	torsocks ssh username@some.host.com -l <user>
or launch pidgin by doing:
	torsocks pidgin

==SECURITY WARNING==
Please note that torsocks does not in general guarantee that all
network connections made using torsocks will always go through
Tor, and not directly through the unsecured network. There are many
ways how general programs can purposefully or inadvertently defeat
torsocks. One way is to reset the environment variable for the child
process. You can use ex. wireshark to see where network packets are
actually sent by the program in question.
```


## C. Конфигурация TOR

Следующий шаг — настройка TOR. По умолчанию файл конфигурации tor находится в папке /usr/local/etc/tor. Отредактируйте файл torrc в этой папке. В файле torrc активируйте несколько опций, удалив знак «#». Следуйте примеру ниже, чтобы включить несколько опций в файле torrc.


```cs
root@ns3:/usr/home/ns3-ghostbsd-pc/Desktop # cd /usr/local/etc/tor
root@ns3:/usr/local/etc/tor # ee torrc
## Configuration file for a typical Tor user
## Last updated 28 February 2019 for Tor 0.3.5.1-alpha.
## (may or may not work for much older or much newer versions of Tor.)
##
## Lines that begin with "## " try to explain what's going on. Lines
## that begin with just "#" are disabled commands: you can enable them
## by removing the "#" symbol.
##
## See 'man tor', or https://www.torproject.org/docs/tor-manual.html,
## for more options you can use in this file.
##
## Tor will look for this file in various places based on your platform:
## https://www.torproject.org/docs/faq#torrc

## Tor opens a SOCKS proxy on port 9050 by default -- even if you don't
## configure one below. Set "SOCKSPort 0" if you plan to run Tor only
## as a relay, and not make any local application connections yourself.
SOCKSPort 192.168.5.4:9050 # Default: Bind to localhost:9050 for local connections.
DNSPort 192.168.5.4:9053                                
AutomapHostsOnResolve 1
## Entry policies to allow/deny SOCKS requests based on IP address.
## First entry that matches wins. If no SOCKSPolicy is set, we accept
## all (and only) requests that reach a SOCKSPort. Untrusted users who
## can access your SOCKSPort may be able to learn about the connections
## you make.
#SOCKSPolicy accept 192.168.0.0/16
#SOCKSPolicy accept6 FC00::/7
#SOCKSPolicy reject *

## Logs go to stdout at level "notice" unless redirected by something
## else, like one of the below lines. You can have as many Log lines as
## you want.
##
## We advise using "notice" in most cases, since anything more verbose
## may provide sensitive information to an attacker who obtains the logs.
##
## Send all messages of level 'notice' or higher to /var/log/tor/notices.log
#Log notice file /var/log/tor/notices.log
## Send every possible message to /var/log/tor/debug.log
#Log debug file /var/log/tor/debug.log
## Use the system log instead of Tor's logfiles
#Log notice syslog
## To send all messages to stderr:
#Log debug stderr

## Uncomment this to start the process in the background... or use
## --runasdaemon 1 on the command line. This is ignored on Windows;
## see the FAQ entry if you want Tor to run as an NT service.
RunAsDaemon 1

## The directory for keeping all the keys/etc. By default, we store
## things in $HOME/.tor on Unix, and in Application Data\tor on Windows.
#DataDirectory /var/db/tor

## The port on which Tor will listen for local connections from Tor
## controller applications, as documented in control-spec.txt.
#ControlPort 9051
## If you enable the controlport, be sure to enable one of these
## authentication methods, to prevent attackers from accessing it.
#HashedControlPassword 16:872860B76453A77D60CA2BB8C1A7042072093276A3D701AD684053EC4C
#CookieAuthentication 1

############### This section is just for location-hidden services ###

## Once you have configured a hidden service, you can look at the
## contents of the file ".../hidden_service/hostname" for the address
## to tell people.
##
## HiddenServicePort x y:z says to redirect requests on port x to the
## address y:z.

#HiddenServiceDir /var/db/tor/hidden_service/
#HiddenServicePort 80 127.0.0.1:80

#HiddenServiceDir /var/db/tor/other_hidden_service/
#HiddenServicePort 80 127.0.0.1:80
#HiddenServicePort 22 127.0.0.1:22

################ This section is just for relays #####################
#
## See https://www.torproject.org/docs/tor-doc-relay for details.

## Required: what port to advertise for incoming Tor connections.
#ORPort 9001
## If you want to listen on a port other than the one advertised in
## ORPort (e.g. to advertise 443 but bind to 9090), you can do it as
## follows.  You'll need to do ipchains or other port forwarding
## yourself to make this work.
#ORPort 443 NoListen
#ORPort 127.0.0.1:9090 NoAdvertise
## If you want to listen on IPv6 your numeric address must be explicitly
## between square brackets as follows. You must also listen on IPv4.
#ORPort [2001:DB8::1]:9050

## The IP address or full DNS name for incoming connections to your
## relay. Leave commented out and Tor will guess.
#Address noname.example.com

## If you have multiple network interfaces, you can specify one for
## outgoing traffic to use.
## OutboundBindAddressExit will be used for all exit traffic, while
## OutboundBindAddressOR will be used for all OR and Dir connections
## (DNS connections ignore OutboundBindAddress).
## If you do not wish to differentiate, use OutboundBindAddress to
## specify the same address for both in a single line.
#OutboundBindAddressExit 10.0.0.4
#OutboundBindAddressOR 10.0.0.5

## A handle for your relay, so people don't have to refer to it by key.
## Nicknames must be between 1 and 19 characters inclusive, and must
## contain only the characters [a-zA-Z0-9].
## If not set, "Unnamed" will be used.
#Nickname ididnteditheconfig

## Define these to limit how much relayed traffic you will allow. Your
## own traffic is still unthrottled. Note that RelayBandwidthRate must
## be at least 75 kilobytes per second.
## Note that units for these config options are bytes (per second), not
## bits (per second), and that prefixes are binary prefixes, i.e. 2^10,
## 2^20, etc.
#RelayBandwidthRate 100 KBytes  # Throttle traffic to 100KB/s (800Kbps)
#RelayBandwidthBurst 200 KBytes # But allow bursts up to 200KB (1600Kb)

## Use these to restrict the maximum traffic per day, week, or month.
## Note that this threshold applies separately to sent and received bytes,
## not to their sum: setting "40 GB" may allow up to 80 GB total before
## hibernating.
##
## Set a maximum of 40 gigabytes each way per period.
#AccountingMax 40 GBytes
## Each period starts daily at midnight (AccountingMax is per day)
#AccountingStart day 00:00
## Each period starts on the 3rd of the month at 15:00 (AccountingMax
## is per month)
#AccountingStart month 3 15:00

## Administrative contact information for this relay or bridge. This line
## can be used to contact you if your relay or bridge is misconfigured or
## something else goes wrong. Note that we archive and publish all
## descriptors containing these lines and that Google indexes them, so
## spammers might also collect them. You may want to obscure the fact that
## it's an email address and/or generate a new address for this purpose.
##
## If you are running multiple relays, you MUST set this option.
##
#ContactInfo Random Person <nobody AT example dot com>
## You might also include your PGP or GPG fingerprint if you have one:
#ContactInfo 0xFFFFFFFF Random Person <nobody AT example dot com>

## Uncomment this to mirror directory information for others. Please do
## if you have enough bandwidth.
#DirPort 9030 # what port to advertise for directory connections
## If you want to listen on a port other than the one advertised in
## DirPort (e.g. to advertise 80 but bind to 9091), you can do it as
## follows.  below too. You'll need to do ipchains or other port
## forwarding yourself to make this work.
#DirPort 80 NoListen
#DirPort 127.0.0.1:9091 NoAdvertise
## Uncomment to return an arbitrary blob of html on your DirPort. Now you
## can explain what Tor is if anybody wonders why your IP address is
## contacting them. See contrib/tor-exit-notice.html in Tor's source
## distribution for a sample.
#DirPortFrontPage /usr/local/etc/tor/tor-exit-notice.html

## Uncomment this if you run more than one Tor relay, and add the identity
## key fingerprint of each Tor relay you control, even if they're on
## different networks. You declare it here so Tor clients can avoid
## using more than one of your relays in a single circuit. See
## https://www.torproject.org/docs/faq#MultipleRelays
## However, you should never include a bridge's fingerprint here, as it would
## break its concealability and potentially reveal its IP/TCP address.
##
## If you are running multiple relays, you MUST set this option.
##
## Note: do not use MyFamily on bridge relays.
#MyFamily $keyid,$keyid,...

## Uncomment this if you want your relay to be an exit, with the default
## exit policy (or whatever exit policy you set below).
## (If ReducedExitPolicy, ExitPolicy, or IPv6Exit are set, relays are exits.
## If none of these options are set, relays are non-exits.)
#ExitRelay 1

## Uncomment this if you want your relay to allow IPv6 exit traffic.
## (Relays do not allow any exit traffic by default.)
#IPv6Exit 1

## Uncomment this if you want your relay to be an exit, with a reduced set
## of exit ports.
#ReducedExitPolicy 1

## Uncomment these lines if you want your relay to be an exit, with the
## specified set of exit IPs and ports.
##
## A comma-separated list of exit policies. They're considered first
## to last, and the first match wins.
##
## If you want to allow the same ports on IPv4 and IPv6, write your rules
## using accept/reject *. If you want to allow different ports on IPv4 and
## IPv6, write your IPv6 rules using accept6/reject6 *6, and your IPv4 rules
## using accept/reject *4.
##
## If you want to _replace_ the default exit policy, end this with either a
## reject *:* or an accept *:*. Otherwise, you're _augmenting_ (prepending to)
## the default exit policy. Leave commented to just use the default, which is
## described in the man page or at
## https://www.torproject.org/documentation.html
##
## Look at https://www.torproject.org/faq-abuse.html#TypicalAbuses
## for issues you might encounter if you use the default exit policy.
##
## If certain IPs and ports are blocked externally, e.g. by your firewall,
## you should update your exit policy to reflect this -- otherwise Tor
## users will be told that those destinations are down.
##
## For security, by default Tor rejects connections to private (local)
## networks, including to the configured primary public IPv4 and IPv6 addresses,
## and any public IPv4 and IPv6 addresses on any interface on the relay.
## See the man page entry for ExitPolicyRejectPrivate if you want to allow
## "exit enclaving".
##
#ExitPolicy accept *:6660-6667,reject *:* # allow irc ports on IPv4 and IPv6 but no more
#ExitPolicy accept *:119 # accept nntp ports on IPv4 and IPv6 as well as default exit policy
#ExitPolicy accept *4:119 # accept nntp ports on IPv4 only as well as default exit policy
#ExitPolicy accept6 *6:119 # accept nntp ports on IPv6 only as well as default exit policy
#ExitPolicy reject *:* # no exits allowed

## Bridge relays (or "bridges") are Tor relays that aren't listed in the
## main directory. Since there is no complete public list of them, even an
## ISP that filters connections to all the known Tor relays probably
## won't be able to block all the bridges. Also, websites won't treat you
## differently because they won't know you're running Tor. If you can
## be a real relay, please do; but if not, be a bridge!
##
## Warning: when running your Tor as a bridge, make sure than MyFamily is
## NOT configured.
#BridgeRelay 1
## By default, Tor will advertise your bridge to users through various
## mechanisms like https://bridges.torproject.org/. If you want to run
## a private bridge, for example because you'll give out your bridge
## address manually to your friends, uncomment this line:
#BridgeDistribution none

## Configuration options can be imported from files or folders using the %include
## option with the value being a path. This path can have wildcards. Wildcards are 
## expanded first, using lexical order. Then, for each matching file or folder, the following 
## rules are followed: if the path is a file, the options from the file will be parsed as if 
## they were written where the %include option is. If the path is a folder, all files on that 
## folder will be parsed following lexical order. Files starting with a dot are ignored. Files 
## on subfolders are ignored.
## The %include option can be used recursively.
#%include /etc/torrc.d/*.conf
```

После настройки TOR создайте файл запуска rc.d, отредактировав файл /etc/rc.conf. Добавьте следующий скрипт в файл /etc/rc.conf.

```cs
root@ns3:/usr/local/etc/tor # ee /etc/rc.conf
tor_enable="YES"
tor_conf="/usr/local/etc/tor/torrc"
tor_user="_tor"
tor_group="_tor"
tor_datadir="/var/db/tor"
```

Перезапустите TOR.

```cs
root@ns3:/usr/local/etc/tor # service tor restart
```

## D. Установка Proxychains

В этом руководстве мы сделаем Proxychains фронтендом и бэкендом TOR. Установка Proxychains на FreeBSD и GhostBSD одинакова. Используйте следующий скрипт для установки Proxychains.

```cs
   root@ns3:/usr/local/etc/tor # pkg install proxychains
Updating GhostBSD repository catalogue...
GhostBSD repository is up to date.
All repositories are up to date.
The following 1 package(s) will be affected (of 0 checked):

New packages to be INSTALLED:
	proxychains: 3.1_3

Number of packages to be installed: 1

20 KiB to be downloaded.

Proceed with this action? [y/N]: y
[1/1] Fetching proxychains-3.1_3.pkg: 100%   20 KiB  21.0kB/s    00:01    
Checking integrity... done (0 conflicting)
[1/1] Installing proxychains-3.1_3...
[1/1] Extracting proxychains-3.1_3: 100%
```

После завершения процесса установки Proxychains продолжите настройку Proxuchains.


## E. Настройка Proxychains

Файл конфигурации или файл конфигурации Proxychains находится в папке /usr/local/etc, отредактируйте файл proxychains.conf. Следуйте сценарию ниже, чтобы настроить файл proxychains.conf.


```cs
root@ns3:/usr/local/etc/tor # cd /usr/local/etc
root@ns3:/usr/local/etc # ee proxychains.conf
# proxychains.conf  VER 3.1
#
#        HTTP, SOCKS4, SOCKS5 tunneling proxifier with DNS.
#	

# The option below identifies how the ProxyList is treated.
# only one option should be uncommented at time,
# otherwise the last appearing option will be accepted
#
dynamic_chain
#
# Dynamic - Each connection will be done via chained proxies
# all proxies chained in the order as they appear in the list
# at least one proxy must be online to play in chain
# (dead proxies are skipped)
# otherwise EINTR is returned to the app
#
#strict_chain
#
# Strict - Each connection will be done via chained proxies
# all proxies chained in the order as they appear in the list
# all proxies must be online to play in chain
# otherwise EINTR is returned to the app
#
#random_chain
#
# Random - Each connection will be done via random proxy
# (or proxy chain, see  chain_len) from the list.
# this option is good to test your IDS :)

# Make sense only if random_chain
#chain_len = 2

# Quiet mode (no output from library)
#quiet_mode

# Proxy DNS requests - no leak for DNS data
proxy_dns 

# Some timeouts in milliseconds
tcp_read_time_out 15000
tcp_connect_time_out 8000

# ProxyList format
#       type  host  port [user pass]
#       (values separated by 'tab' or 'blank')
#
#
#        Examples:
#
#            	socks5	192.168.67.78	1080	lamer	secret
#		http	192.168.89.3	8080	justu	hidden
#	 	socks4	192.168.1.49	1080
#	        http	192.168.39.93	8080	
#		
#
#       proxy types: http, socks4, socks5
#        ( auth types supported: "basic"-http  "user/pass"-socks )
#
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
#socks4 	192.168.5.4 9050
socks5  192.168.5.4 9050
```

Постарайтесь обратить пристальное внимание на все скрипты proxychains.conf, в нижней строке написано "socks5 192.168.5.4 9050", что означает, что прокси-сеть из Proxychains взята из TOR, потому что IP 192.168.5.4 и порт 9050 - это IP и порт TOR. Таким образом, ясно, что TOR - это бэкэнд Proxychains.


## F. Тестирование и запуск Proxychains

Если вы не пропустили ни одного из шагов выше, вы можете перейти к следующему шагу, а именно, к выполнению или запуску Proxychain. В отличие от процесса установки выше, для запуска Proxychain вы должны войти в систему как гостевой пользователь. В этом руководстве я создал гостевого пользователя с именем "ns3-ghostbsd-pc". Чтобы запустить Proxychain, мы входим в систему как пользователь ns3-ghostbsd-pc, вводим следующий скрипт.

```cs
root@ns3:/usr/home/ns3-ghostbsd-pc/Desktop # su ns3-ghostbsd-pc
Welcome to fish, the friendly interactive shell
Type help for instructions on how to use fish
ns3-ghostbsd-pc@ns3 /u/h/n/Desktop> 
```

В первом скрипте, который отмечен красным, вы активны в качестве пользователя root, после ввода "su ns3-ghostbsd-pc" и нажатия Enter вы будете в качестве гостевого пользователя с именем ns3-ghostbsd-pc. После активации в качестве пользователя ns3-ghostbsd-pc введите следующий скрипт для запуска Proxychains.

```cs
root@ns3:/usr/home/ns3-ghostbsd-pc/Desktop # proxychains firefox yandex.ru
```

Скрипт выше откроет веб-сайт yandex.ru. Если с конфигурацией выше все в порядке, браузер Modzilla Firefox немедленно откроет веб-сайт yandex.ru.

Использование Proxychains в частной сети, очевидно, очень выгодно, поскольку повышает безопасность и конфиденциальность. Если вы хотите испытать качественные услуги Proxychains с несколькими прокси в одной цепочке, используйте Proxychains. Если вы хотите оставаться анонимным при просмотре данных в сети, используйте TOR в качестве бэкэнда Proxychains. Сочетание TOR и Proxychains очень подходит для высокоуровневых приложений, таких как скрапинг. Для скрытия физических IP-адресов и анонимизации данных в нашей компьютерной сети.
