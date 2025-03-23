---
title: Checking the Status of Open Ports on Windows and FreeBSD
date: "2025-01-19 15:01:10 +0100"
id: checking-status-open-portson-windows-and-freebsd
lang: en
categories:
  - FreeBSD
tags: "freebsd, openbsd, linux. ubuntu, windows"
excerpt: You can scan the network system and find out which computers have open ports.
keywords: freebsd, port, netstat, nc, command, status, open, windows
layout: single
author_profile: true
---

In this article we will not explain any more, the importance of noting open ports on your computer system. If you are looking for a way to find out which ports are wide open on the computer you are using, you are on the right blog. Keep reading the contents of this article, until you understand how to read the currently open ports.

The biggest problem for a network system administrator is open ports on devices. Even if you don't install the applications on these ports, if left unchecked, you will be at risk of an attack that will damage your computer system.

You can scan the network system and find out which computers have open ports. When you know the location of open ports, you can check the information against your master list and immediately take action to close all unnecessary ports.

Port scanning is carried out to determine which ports of computers on your network are open. Through open ports, we can find possible vulnerabilities in the target host and fix them in time. Therefore, host port scanning can help us better understand the target host and is the first step to properly strengthen security.

By adopting TCP full connection technology we can achieve port scanning. The scanning host tries (using a TCP three-way handshake) to establish a regular connection with the designated port of the destination host, as shown in the following Figure.

In this tutorial, you will learn how to check for open ports in Windows and FreeBSD operating systems using a handy command line tool that you can run in the Command Line shell.<br><br/>
## 1. View Open Windows ports
Netstat is a general purpose command line utility included with the Windows operating system. Its functions include displaying the network status with all the necessary details. Users use the built-in syntax to filter results or define additional actions available in the Netstat utility. The content of this article will also describe in detail the available methods for viewing open ports using the Netstat utility.

Since Netstat is one of the console command applications, you need to run "CMD" to run it. Open the Start menu, find "Command Prompt" there, and launch it.

### a. Displays all connections
To see the entire list of open ports in Windows use the "netstat -a" command in the CMD menu. The command will display all information about active connections from the ports that are currently open. This information is available for viewing without administrator rights and is displayed as follows:

C:\Users\opensource>**netstat -a**

### b. Identify Open Ports
Netstat is short for network statistics. By using Netstat we can see TCP and IP network protocol and connection statistics on the server or client computer that is being used. Use the netstat -ab and netstat -aon commands to identify open ports.

Below is the meaning and explanation of the command
- a =  Displays information about all connections.
- b =  shows all executables involved in creating each listening port.
- o =  provides the owning process ID related to each connection.
- n = shows the addresses and port numbers as numerals.
- more= Paginated output of elements

In the CMD shell menu, run the following command.

```
C:\Users\blogspot>netstat -ab
C:\Users\blogspot>netstat -aon
```

You can also use the command "netstat -aon | more" to display pagination of open ports.

```
C:\Users\blogspot>netstat -aon | more
```

### c. Search by content
Other Netstat capabilities can display connections with certain parameters or addresses, for example only displaying listening ports.

```
C:\Users\blogspot>netstat -a | find /I "LISTENING"
```

### d. Write results to a text file
Another Netstat command is that it can save completed monitoring results to a text file. Run the following command to get the output in "txt" format.

```
C:\Users\blogspot>netstat -aon | more > netstat.txt
```
<br><br/>
## 2. View open FreeBSD ports
If you are using a BSD operating system, such as FreeBSD or others, the Netstat command can also be used to find out which TCP/UDP ports can be listened to or opened, network connections, routing tables, interface statistics, masked connections, and multicast membership.

### a. View connection status
You can use the Netstat command to view the TCP/UDP connection status. Run the following command to display TCP/UDP status.

```
root@ns3:~ # netstat -a -p tcp
root@ns3:~ # netstat -a -p udp
```

### b. View all sockets in the Listening state
```
root@ns3:~ # netstat -a | egrep 'Proto|LISTEN'
root@ns3:~ # netstat -a -p tcp | egrep 'Proto|LISTEN'
root@ns3:~ # netstat -a -p udp | egrep 'Proto|LISTEN'
```

### c. Display statistics for each protocol

```
root@ns3:~ # netstat -s
```

### d. View Open TCP and UDP Ports

```
root@ns3:~ # netstat -an | egrep "^tcp|^udp"
```

After you read this article, you need to remember that using the Netstati utility does not eliminate the use of a personal firewall to protect the system, but rather complements it. You can fine-tune firewall rules based on data obtained from the NEtstat utility. For example, what ports are involved, and the addresses to which the program sends traffic. You can block applications, ports or addresses that we don't like.
