---
title: Build Instructions OBS Studio For GhostBSD Desktop FreeBSD
date: "2025-01-18 14:11:10 +0300"
id: build-instructions-obs-studio-freebsd
lang: en
layout: single
categories:
  - FreeBSD
tags: "SysAdmin"
excerpt: OBS Studio also supports many plugins that can expand its functionality to include features such as NDI support
keywords: freebsd, obs, studio, building, ghostbsd, desktop, instruction
---

OBS stands for Open Broadcaster Software, a free, open source live video production software supported by a large community of developers from around the world. OBS can be used for live video production, live streaming, and video recording. When you first download and install OBS, the setup wizard will ask if you want to optimize the software for recording or live streaming because OBS has the ability to combine many different audio visual sources into a live video production environment.

OBS Studio also supports many plugins that can expand its functionality to include features such as NDI support, VST plugins, and stream deck control. OBS Studio plugins are software add-ons that are installed on the program to enable more features. In the case of OBS Studio, users have to install the plugin manually.

Plugins in OBS Studio are usually new sources in the sources list. However, some of them may even install an entirely new tab or dock to your program. It completely depends on the functionality of the plugin.

The plugin function in OBS Studio is used to extend OBS functionality by adding custom code written to perform specific tasks. The most popular OBS plugins add support for NDI, which is an IP video production protocol. Another popular plugin is called VirtualCam, which lets you capture any video within OBS and connect it to another camera via a virtual webcam source.

Typically independent developers identify areas that can be improved within OBS Studio and code solutions for them. They then host the files for their new software online, and OBS users can download and install them into their OBS Studio directory.

Who uses OBS Studio. gamers, content creators, and online educators regularly use software to record content such as gameplay, tutorials, vlogs, and more. The program can also broadcast live video to popular streaming platforms and provides the option to customize the look and feel of the stream with themes, transitions, and other elements.

On this occasion, we will try to discuss how to install OBS Studio on a UNIX BSD based system. In this case we will try to install OBS Studio into the FReeBSD Desktop System. Because there are so many FreeBSD Desktop variants, this article will explain how to install OBS Studio on a GhostBSD system, one of the Desktop-based FreeBSD variants with a charming GUI display.

## 1. System Specifications
- OS: GhostBSD 23.06.01
- CPU: Intel(R) Core(TM)2 Quad CPU Q9550 @ 2.83GHz
- Memory: 4 GB
- OBS version: obs-studio-29.1.3_2

## 2. OBS Studio Dependencies
Dependencies are library files required to install OBS Studio. This library file is very important for a program to run perfectly on the FreeBSD system. There are several dependencies that must be installed before you install OBS Studio. The following are the dependencies that must be installed on GhostBSD.

```
root@ns3:~ # pkg install cmake ninja pkgconf curl ccache
Updating GhostBSD repository catalogue...
GhostBSD repository is up to date.
All repositories are up to date.
The following 4 package(s) will be affected (of 0 checked):

New packages to be INSTALLED:
	ccache: 3.7.12_4
	ninja: 1.11.1,2

Installed packages to be UPGRADED:
	libunibreak: 4.3,1 -> 5.1,1
	pkgconf: 1.8.0_1,1 -> 1.8.1,1

Number of packages to be installed: 2
Number of packages to be upgraded: 2

241 KiB to be downloaded.

Proceed with this action? [y/N]: y
[1/2] Fetching ccache-3.7.12_4.pkg: 100%  128 KiB 131.4kB/s    00:01    
[2/2] Fetching ninja-1.11.1,2.pkg: 100%  113 KiB 115.3kB/s    00:01    
Checking integrity... done (0 conflicting)
[1/4] Installing ccache-3.7.12_4...
[1/4] Extracting ccache-3.7.12_4: 100%
Create compiler links...
create symlink for cc
create symlink for cc (world)
create symlink for c++
create symlink for c++ (world)
create symlink for CC
create symlink for CC (world)
create symlink for gcc
create symlink for gcc (world)
create symlink for g++
create symlink for g++ (world)
create symlink for gcc12
create symlink for gcc12 (world)
create symlink for g++12
create symlink for g++12 (world)
create symlink for cpp12
create symlink for cpp12 (world)
create symlink for clang
create symlink for clang (world)
create symlink for clang++
create symlink for clang++ (world)
create symlink for clang15
create symlink for clang15 (world)
create symlink for clang++15
create symlink for clang++15 (world)
[2/4] Installing ninja-1.11.1,2...
[2/4] Extracting ninja-1.11.1,2: 100%
[3/4] Upgrading libunibreak from 4.3,1 to 5.1,1...
[3/4] Extracting libunibreak-5.1,1: 100%
[4/4] Upgrading pkgconf from 1.8.0_1,1 to 1.8.1,1...
[4/4] Extracting pkgconf-1.8.1,1: 100%
=====
Message from ccache-3.7.12_4:

--
NOTE:
Please read /usr/local/share/doc/ccache/ccache-howto-freebsd.txt for
information on using ccache with FreeBSD ports and src.
```

Next, there are still more dependencies that you have to install.

```
root@ns3:~ # pkg install ffmpeg libx264 mbedtls mesa-libs jansson lua52 luajit python39 libX11 xorgproto libxcb libXcomposite libXext libXfixes libXinerama libXrandr swig dbus jansson libICE libSM libsysinfo
Updating GhostBSD repository catalogue...
GhostBSD repository is up to date.
All repositories are up to date.
The following 5 package(s) will be affected (of 0 checked):

New packages to be INSTALLED:
	lua52: 5.2.4
	luajit: 2.0.5_6
	swig: 4.1.1

Installed packages to be UPGRADED:
	mbedtls: 2.28.2 -> 2.28.3
	xorgproto: 2022.1 -> 2022.1_1

Number of packages to be installed: 3
Number of packages to be upgraded: 2

The process will require 23 MiB more space.
6 MiB to be downloaded.

Proceed with this action? [y/N]: y
[1/2] Fetching lua52-5.2.4.pkg: 100%  175 KiB 179.0kB/s    00:01    
[2/2] Fetching swig-4.1.1.pkg: 100%    6 MiB 241.5kB/s    00:24    
Checking integrity... done (2 conflicting)
  - luajit-2.0.5_6 conflicts with luajit-devel-2.1.0.20230712 on /usr/local/bin/luajit
  - luajit-2.0.5_6 conflicts with luajit-devel-2.1.0.20230712 on /usr/local/bin/luajit
Checking integrity... done (0 conflicting)
Conflicts with the existing packages have been found.
One more solver iteration is needed to resolve them.
The following 8 package(s) will be affected (of 0 checked):

Installed packages to be REMOVED:
	luajit-devel: 2.1.0.20230712
	obs-studio: 29.1.3_2

New packages to be INSTALLED:
	lua52: 5.2.4
	luajit: 2.0.5_6
	swig: 4.1.1

Installed packages to be UPGRADED:
	mbedtls: 2.28.2 -> 2.28.3
	xorgproto: 2022.1 -> 2022.1_1

Installed packages to be REINSTALLED:
	libXxf86dga-1.1.6

Number of packages to be removed: 2
Number of packages to be installed: 3
Number of packages to be upgraded: 2
Number of packages to be reinstalled: 1

The operation will free 9 MiB.

Proceed with this action? [y/N]: y
[1/8] Deinstalling luajit-devel-2.1.0.20230712...
[1/8] Deleting files for luajit-devel-2.1.0.20230712: 100%
[2/8] Deinstalling obs-studio-29.1.3_2...
[2/8] Deleting files for obs-studio-29.1.3_2: 100%
[3/8] Upgrading xorgproto from 2022.1 to 2022.1_1...
[3/8] Extracting xorgproto-2022.1_1: 100%
[4/8] Installing luajit-2.0.5_6...
[4/8] Extracting luajit-2.0.5_6: 100%
[5/8] Reinstalling libXxf86dga-1.1.6...
[5/8] Extracting libXxf86dga-1.1.6: 100%
[6/8] Installing lua52-5.2.4...
[6/8] Extracting lua52-5.2.4: 100%
[7/8] Installing swig-4.1.1...
[7/8] Extracting swig-4.1.1: 100%
[8/8] Upgrading mbedtls from 2.28.2 to 2.28.3...
[8/8] Extracting mbedtls-2.28.3: 100%
==> Running trigger: gtk-update-icon-cache.ucl
Generating GTK icon cache for /usr/local/share/icons/hicolor
==> Running trigger: desktop-file-utils.ucl
Building cache database of MIME types
```

## 3. OBS Studio installation
After the above dependencies are installed, we continue by installing the main program, namely OBS Studio. To install OBS Studio on Ghost BSD you must log in as a root user. Follow these steps to install OBS Studio.

```
root@ns3:~ # cd /usr/ports/ports-mgmt/portmaster
root@ns3:/usr/ports/ports-mgmt/portmaster # make install clean
root@ns3:/usr/ports/ports-mgmt/portmaster # cd /usr/ports/ports-mgmt/portupgrade
root@ns3:/usr/ports/ports-mgmt/portupgrade # make install clean
```

The command above is used to prepare [updates and upgrades to the ports system](https://www.inchimediatama.org/2025/02/update-paket-pkg-freebsd-14-dengan.html) on GhosBSD. After both programs are installed, continue by updating the GhostBSD system ports.

```
root@ns3:/usr/ports/ports-mgmt/portupgrade # portmaster -af
root@ns3:/usr/ports/ports-mgmt/portupgrade # portupgrade -af
```

The portupgrade -af and portmaster -af scripts are for updating the GhostBSD ports system. After completing the ports update, we can immediately install OBS Studio. The following are the commands to install OBS Studio.

```
root@ns3:/usr/ports/ports-mgmt/portupgrade # cd /usr/ports/multimedia/obs-studio
root@ns3:/usr/ports/multimedia/obs-studio # make install clean
```

On GhostBSD, sometimes the installation using the ports system is unsuccessful or errors often occur. If the installation fails with the ports system, we can try installing OBS Studio with the FreeBSD pkg package. Here's how to install it.

```
root@ns3:/usr/ports/multimedia/obs-studio # pkg install obs-studio
Updating GhostBSD repository catalogue...
GhostBSD repository is up to date.
All repositories are up to date.
Checking integrity... done (2 conflicting)
  - luajit-devel-2.1.0.20230712 conflicts with luajit-2.0.5_6 on /usr/local/bin/luajit
  - luajit-devel-2.1.0.20230712 conflicts with luajit-2.0.5_6 on /usr/local/bin/luajit
Checking integrity... done (0 conflicting)
The following 3 package(s) will be affected (of 0 checked):

Installed packages to be REMOVED:
	luajit: 2.0.5_6

New packages to be INSTALLED:
	luajit-devel: 2.1.0.20230712
	obs-studio: 29.1.3_2

Number of packages to be removed: 1
Number of packages to be installed: 2

The process will require 30 MiB more space.

Proceed with this action? [y/N]: y
[1/3] Deinstalling luajit-2.0.5_6...
[1/3] Deleting files for luajit-2.0.5_6: 100%
[2/3] Installing luajit-devel-2.1.0.20230712...
[2/3] Extracting luajit-devel-2.1.0.20230712: 100%
[3/3] Installing obs-studio-29.1.3_2...
[3/3] Extracting obs-studio-29.1.3_2: 100%
==> Running trigger: gtk-update-icon-cache.ucl
Generating GTK icon cache for /usr/local/share/icons/hicolor
==> Running trigger: desktop-file-utils.ucl
Building cache database of MIME types
```

After OBS Studio is installed, we continue by typing the following command in the /etc/rc.conf file.

```
root@ns3:~ # ee /etc/rc.conf
kld_list="cuse4bsd"
webcamd_enable="YES"
```

The command above is used to activate the webcam plugin, so that OBS Studio can record using the webcam.


## 4. How to Run OBS Studio
On GhsotBSD, running OBS Studio is very easy, on the GhostBSD Desktop display, right click the mouse and select the "Open in Terminal" menu. After entering the Ghost BSD command shell menu, type the command "obs" followed by the "enter" key on the keyboard.

```
semeru@ns3:/usr/home/semeru/Desktop $ obs
```

Then the OBS Studio program will appear on your monitor screen.

OBS Studio is a great program for advanced users. The program can be configured to suit your needs as a recording program and even create professional-level final products for sites like YouTube and Twitch.



