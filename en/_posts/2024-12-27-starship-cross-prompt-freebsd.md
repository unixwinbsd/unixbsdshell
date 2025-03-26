---
title: Configuration Starship cross shell prompt For FreeBSD
date: "2024-12-27 11:22:10 +0200"
id: starship-cross-prompt-freebsd
lang: en
layout: single
author_profile: true
categories:
  - FreeBSD
tags: "UnixShell"
excerpt: Starship includes free open source tools that you can download from the Github repository or install directly via the FreeBSD server
keywords: starship, freebsd, cross, shell, prompt, command
---

Starship runs with shell commands and is built in the rust language. This utility has minimal prompts, is very fast, simple and highly customizable for any shell. Each command on Starship displays the information you need while you work, but remains neat and does not interfere with your work on the monitor screen.

Starship includes free open source tools that you can download from the Github repository or install directly via the FreeBSD server. Starship features autocomplete commands, syntax highlighting, cross-shell prompts, etc.

With its ease of use and rich features, Starship can enhance your working experience with the terminal, and you feel refreshed with a fantastic terminal. Starship works with every shell, such as bash, zsh (z shell), Nushell, PowerShell, etc.

It seems that Starship doesn't care about the shell you use, whether on the FreeBSD server you use bash, fish or another shell, you can use Starship to adjust its appearance.

What you need to pay attention to when using Starship, you have to read the official documentation to be able to carry out advanced configurations for everything you like. In this article we just explain a simple configuration to get started along with some important information about Startship.<br><br/>
## 1. Install Starship Shell Prompt
FreeBSD makes it easy to install Starship, because you don't need to download the Starship file. The Starship repository is available in the PKG package and the FreeBSD system port, and you can install Starship directly. Please remember, before you start installing Starship, install the Starship dependencies first. The following are the dependencies that you must install.

```
root@ns3:~ # pkg install rust cmake-core libgit2 libssh2
```

To make it clearer, below is a guide to installing Starship with the FreeBSD port system.

```
root@ns3:~ # cd /usr/ports/shells/starship
root@ns3:/usr/ports/www/mod_wsgi4 # make install clean
```

After you type "make install clean", wait a few moments until the installation process is complete.<br><br/>
## 2. Starship Configuration
On FreeBSD using Starship is very easy, you can follow the instructions provided. Run the help command to display the Starship usage guide.

```
root@ns3:~ # starship -help
The cross-shell prompt for astronauts.

Usage: starship <COMMAND>

Commands:
  bug-report    Create a pre-populated GitHub issue with information about your configuration
  completions   Generate starship shell completions for your shell to stdout
  config        Edit the starship configuration
  explain       Explains the currently showing modules
  init          Prints the shell function used to execute starship
  module        Prints a specific prompt module
  preset        Prints a preset config
  print-config  Prints the computed starship configuration
  prompt        Prints the full starship prompt
  session       Generate random session key
  timings       Prints timings of all active modules
  toggle        Toggle a given starship module
  help          Print this message or the help of the given subcommand(s)

Options:
  -h, --help     Print help
  -V, --version  Print version
```

After that, you create a special Starship directory. By default FreeBSD, the Starship directory is /root/.cache/starship. Let's just use the existing Starship directory. So we just create a toml file.

```
root@ns3:~ # cd /root/.cache/starship
root@ns3:~/.cache/starship # ee starship.toml

[cmd_duration]
disabled = true

[directory]
truncation_symbol = "…/"

[nodejs]
disabled = true

[package]
disabled = true

[php]
format = "PHP: [$version](147 bold) "

[rust]
disabled = true
```

If your FreeBSD server has bash shell installed, open the “~/.bashrc” file and paste this line of code at the end.

```
root@ns3:~ # ee .bashrc
eval "$(starship init bash)"
```

After that, run the command below.

```
root@ns3:~ # starship print-config | tee /root/.cache/starship/starship.toml
```

Restart your FreeBSD server.

```
root@ns3:~ # reboot
```

The configuration examples above are just the basics of using Starship on FreeBSD, but there's a lot more you can do with Starship by following their online manual.

