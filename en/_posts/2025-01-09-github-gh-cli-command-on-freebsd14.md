---
title: Guide to Using Github GH CLI Commands on FreeBSD 14
date: "2025-02-06 09:17:10 +0300"
id: github-gh-cli-command-on-freebsd14
lang: en
layout: single
author_profile: true
categories:
  - FreeBSD
tags: "SysAdmin"
excerpt: GH is a Github product that is a Github command line tool that brings pull requests, issues, and other GitHub concepts
keywords: github, gh, cli, command, freebsd14, github pages, tools, utility
---

GitHub is a code hosting provider, which can be used for version control and collaboration. With Github it can make it easier for you to work together in a team, or with other people on the same project. Additionally, GitHub can be a Git hosting repository providing tools for developers to submit better code using command line features, issues (threaded discussions), pull requests, code reviews, or using the selection of free and paid apps in the GitHub Marketplace.  

If you're already familiar with FreeBSD's Git commands, it's time to switch to the web browser's CLI commands to perform various actions on your GitHub repository. With the new GitHub GH CLI tool, you can run many commands without leaving the command line interface.  

GH is a Github product that is a Github command line tool that brings pull requests, issues, and other GitHub concepts to the terminal next to where you work with git and your code. GH will reduce context switching, help you focus, and allow you to script and create your own workflows more easily, quickly and efficiently.  

The guide in this article will explain the steps for installing GitHub GH CLI on a FreeBSD server. GitHub CLI (GH) is a command line GitHub feature tool. This tool brings pull requests, issues, and other GitHub concepts to the terminal next to where you work with git and your code.

![Github GH CLI command](https://gitflic.ru/project/iwanse1212/unixwinbsd/blob/raw?file=Github%20GH%20CLI%20command.jpg)

## 1. Install GH CLI
Anyone can install the Github CLI, even if they are new to the FreeBSD system. Github provides easy installation of its own application, namely "GH". Here's how to install Github GH CLI on FreeBSD.

```
root@ns3:~ # cd /usr/ports/devel/gh root@ns3:/usr/ports/devel/gh # make config root@ns3:/usr/ports/devel/gh # make install clean
```

After installing GitHub CLI, you should check the GH CLI version with the following command.

```
root@ns3:~ # gh --version
gh version 2.39.1 (2024-01-30)
https://github.com/cli/cli/releases/tag/v2.39.1
```

## 2. Authenticate Github GH CLI
Github offers 2 Github GH CLI Authentication options, namely:

1.  HTTPS, and
2.  SSH

In this article we will discuss both, so that readers can choose one of the two authentication methods.

### a. Authenticate HTTPS
The next step, authenticate your Github account. Run the command "gh auth login" to authenticate your GitHub account. It will ask you to select the account type, either a regular account or a corporate account. I will choose a normal account.

```
root@ns3:~ # gh auth login
? What account do you want to log into? GitHub.com
? What is your preferred protocol for Git operations? HTTPS
? How would you like to authenticate GitHub CLI? Login with a web browser

! First copy your one-time code: 81D1-CEE8
Press Enter to open github.com in your browser...
! Failed opening a web browser at https://github.com/login/device
  exec: "xdg-open": executable file not found in $PATH - install xdg-utils from ports(8)
  Please try entering the URL in your browser manually
? Authentication complete.
- gh config set -h github.com git_protocol https
? Configured git protocol
! Authentication credentials saved in plain text
? Logged in as iwanse1977
root@ns3:~ #
```

Then the authentication method display will appear. You must select your preferred authentication method. You can choose to authenticate using your browser or using a GitHub authentication token. In this article we will use the browser method.

Then an authentication code will appear. You have to copy the code in a web browser, you have to log in to your Github account, and type the following URL  **"https://github.com/login/device"**. Look at the blue writing above and the image below.

![github Device Activation](https://gitflic.ru/project/iwanse1212/unixwinbsd/blob/raw?file=github%20Device%20Activation.jpg)

### b. Authenticate SSH
The main requirement for SSH Authentication is that you must have an SSH public key, how to do it is easy, you can read our previous article about creating public keys for Github.

**[Setup Github Via FreeBSD SSH To Upload Repository from Local Computer](https://www.blogger.com/u/1/blog/post/edit/3047631139734470358/663489472755747923#)**  

Once you have the SSH public key, you can run the command "gh auth login" in the "What is your preferred protocol for GIT operation?" you select SSH.

If you choose the SSH authentication method, the following options will appear:  

How would you like to authenticate GitHub CLI? [Use arrows to move, type to filter]

> Login with a web browser
> 
> Paste an authentication token  

Choose one of the two options, for example you would choose "Paste an authentication token". You must create a token on your Github account. The method:

1. Click the settings menu

2. Click "Developer settings" at the bottom right of the "Public profile" menu.

3. Click the "Personal access token" menu

4. Click "Fine-grained tokens"

5. Click generate new token

6. After you have filled in all the columns, click "Generate token"

![Github GH CLI Generate token](https://gitflic.ru/project/iwanse1212/unixwinbsd/blob/raw?file=Github%20GH%20CLI%20Generate%20token.jpg)

After you get the token as shown above, copy the token to your Putty terminal.

## 3. How to Use GH CLI
The gh command structure is in the form of a tree, making it easier for users to remember. The GH command has two basic command levels. The first level consists of only six commands yaitu config, repo, issue, pr, gist, credits.

Below we will provide some examples of basic usage of Github GH CLI commands.

**a. gh authorization logout***

```
root@ns3:~ # gh auth logout
? Logged out of github.com account 'jhondoe123'
```

**b. updating gh authorization**

Update the currently active Authentication Credentials.

```
root@ns3:~ # gh auth refresh --scopes write:org,read:public_key
! Using secure storage could break installed extensions
! First copy your one-time code: C568-2A66
Press Enter to open github.com in your browser...
```

**c. gh authorization status**

```
root@ns3:~ # gh auth status
github.com
  ? Logged in to github.com as iwanse1977 (/root/.config/gh/hosts.yml)
  ? Git operations for github.com configured to use ssh protocol.
  ? Token: gho_************************************
  ? Token scopes: gist, read:public_key, repo, write:org
```

**d. Cloning Repository**

```
root@ns3:/tmp # gh repo clone https://github.com/unixwinbsd/Go-ReverseProxy-FreeBSD.git
```  

**e. Create Repository**

```
root@ns3:~ # gh repo create nodejs-webcrawler --public --source=/usr/local/etc/crawling/nodejs-webcrawler
```
  
**f. Delete Repository**
Before you run the delete command, first run the following command.

```
root@ns3:~ # gh auth refresh -h github.com -s delete_repo
! Using secure storage could break installed extensions
! First copy your one-time code: A9C7-1BDD
Press Enter to open github.com in your browser...
```

Pay attention to the blue writing, open "Google Chrome" type the following command  **"https://github.com/login/device"**. You enter the blue code in the Google Chrome display of your GitHub account.

After that, continue the delete repository command.

```
root@ns3:~ # gh repo delete nodejs-webcrawler
? Type  jhondoe123/nodejs-webcrawler  to confirm deletion:  jhondoe123/nodejs-webcrawler
? Deleted repository jhondoe123/nodejs-webcrawler
```

Github GH CLI is a great utility created by Github developers. This utility is very useful for a system administrator who enjoys working in a shell terminal.
