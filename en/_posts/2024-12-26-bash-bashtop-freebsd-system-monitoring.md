---
title: Configuration Bashtop - System Monitoring resources On FreeBSD 14
date: "2024-12-26 15:11:10 +0100"
id: bash-bashtop-freebsd-system-monitoring
lang: en
layout: single
author_profile: true
categories:
  - FreeBSD
tags: "UnixShell"
excerpt: Bashtop is written in Bash with a TUI interface to monitor resource usage on your Linux, macOS, or FreeBSD system
keywords: bash, freebsd, command, utility, bashtop, shell, unix
---
Bashtop is a free and open source resource monitoring tool for FreeBSD, macOS, Linux, and Unix systems. This tool can display CPU status, memory usage, hard disk usage, network, and other running process usage and statistics.

Bashtop is written in Bash with a TUI interface to monitor resource usage on your Linux, macOS, or FreeBSD system. Bashtop has a customizable menu display and a fully responsive terminal user interface.

A newer version of bashtop is now available written in the Python programming language called bpytop. This version of Bashtop CLI is much faster and consumes only about 1/3 of the resources of native bashtop. Bpytop is less CPU intensive and includes more features, such as:

1.  Easy to use, with a menu display like an online game.
2.  UI menu to change all configuration file options.
3.  Shows the current disk read and write speed.
4.  Mouse support.
5.  Send SIGTERM, SIGKILL, SIGINT to the selected process.
6.  Changeable mini mode.
7.  Function to display detailed statistics for the selected process.
8.  Displays a message in the menu if a new version is available.
9.  Additional customization.
10.  Custom graphics for memory consumption.
11.  Fast and “mostly” responsive UI with UP, DOWN button process selection.
12.  Ability to filter processes.
13.  Easy switching between sorting options.
14.  Auto scaling graph for network usage.

Using Bashtop, you can sort processes and easily switch between various sorting options. Additionally, Bashtop can also send SIGKILL, SIGTERM and SIGINT to the required processes. This tool runs very fast and perfect system monitoring. This tool reaches users with a responsive terminal interface with a customizable menu. Monitoring various system metrics is made easy by the neat arrangement of the various display sections.<br><br/>
## 1. Installing Bashtop on FreeBSD
To install Bashtop on FreeBSD, you make sure Bash is installed on your FreeBSD server. If not, run the following command to install Bash.

```
root@ns7:~ # cd /usr/ports/shells/bash
root@ns7:/usr/ports/shells/bash # make install clean
```

Apart from Bash dependencies, you also have to install other dependencies, such as coreutils, gsed and python39. Here's how to install these dependencies.

```
root@ns7:~ # pkg install python39 coreutils gsed
```

Installing Bashtop on FreeBSD is very easy without any special configuration. How to run Bashtop is also very simple, so it's natural that this application is so light and fast. Run the following command to install Bashtop.

```
root@ns7:~ # cd /usr/ports/sysutils/bashtop
root@ns7:/usr/ports/sysutils/bashtop # make install clean
```

After installation, when we want to start Bashtop, you simply run the following command in the FreeBSD terminal.

```
root@ns7:~ # bashtop
```

If there is nothing wrong with the installation, the command above will display the following image.

![bashtop system monitoring tool on freebsd](https://www.opencode.net/unixbsdshell/freebsd-golang-mysql-crud/-/raw/main/template/bashtop_system_monitoring_tool_on_freebsd.jpg)

Bashtop provides computer users with a great way to keep an eye on system resources. With Bashtop you can monitor computer resources very easily and with a stunning game display.
