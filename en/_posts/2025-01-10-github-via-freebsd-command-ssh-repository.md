---
title: Setup Github Via FreeBSD SSH To Upload Repository from Local Computer
date: "2025-01-12 08:17:10 +0100"
id: github-via-freebsd-command-ssh-repository
lang: en
layout: single
author_profile: true
categories:
  - FreeBSD
tags: "UnixShell"
excerpt: Git is a free open source project, so many large software projects depend on Git for version control
keywords: github, gh, cli, command, freebsd14, github pages, ssh, openssh
---

Control systems are the most important part of the computer world. With a control system, each user can change, delete or add files to the server. Because of its importance, in 2005, Linus Torvalds, the creator of the famous Linux operating system kernel created a control system called "GIT".

Git is a free open source project, so many large software projects depend on Git for version control, including both commercial and open source projects. Developers who have worked with Git are well represented in the available software development talent pool and work well on a variety of operating systems and IDEs (Integrated Development Environments).

In addition to being distributed free of charge, Git has been designed with performance, security, and flexibility in mind.

**a. Flexibility**
One of the goals of Git's design was flexibility. Git is very flexible in several ways, especially in supporting various types of non-linear development workflows, in its efficiency in both large, small and medium projects. Its compatibility reliability is supported by many existing systems and protocols.

**b. Performance**
When compared to other similar applications, Git's raw performance characteristics are very strong. GIT can make changes quickly, create branches, merge, and compare previous versions all optimized for performance. The algorithms implemented within Git leverage deep knowledge of the general attributes of the actual source code file tree, how they are typically modified over time, and what their access patterns are.

GIT's performance is very reliable, because Git will not be fooled by file names when determining what storage and file tree version history should be, instead, Git focuses on the contents of the file itself. After all, source code files are frequently renamed, split, and rearranged. The Git repository file object format uses a combination of delta encodings (saving content differences).  

**c. Security**
From the beginning, Git has been designed with managed open source code integrity as a top priority. The contents of files as well as the actual relationships between files and directories, versions, tags and deployments, all objects in this Git repository are secured with a cryptographically secure hashing algorithm called SHA1. This protects the code and change history against accidental and malicious changes and ensures that the history is fully traceable.
<br><br/>
## 1. GIT installation
Before we start uploading files from the local computer to the GitHub server, make sure GIT is installed on your FreeBSD. Below are the commands you can use to install GIT on FreeBSD.

There are two ways to install Git on FreeBSD:
1.  FreeBSD package PKG, and
2.  FreeBSD system ports

Before installing Git, make sure the PKG package is updated.

```
root@ns7:~ # pkg update -f
root@ns7:~ # pkg upgrade -f
```

Run the following command to install Git.

```
root@ns7:~ # pkg install git
```

The second way is with the port system, ports are a collection of more than 20,000 third-party programs that are available for free on FreeBSD. The installation process is much slower because the process involves building, that is, compiling the package from source code.

Follow the steps below to install Git on FreeBSD using the ports system.

```
root@ns7:~ # cd /usr/ports/devel/git
root@ns7:/usr/ports/devel/git # make install clean
```
<br><br/>
## 2. Enable SSH Keygen

If you want to have a private repository on GitHub and want other people to read it, we need an SSH key to connect to the Github server. Actually there are many ways to connect to the Github server, SSH is the easiest and simplest way.

You can easily create a public SSH key with the following command.

```
root@ns7:~ # ssh-keygen -t ed25519 -C "datainchi@gmail.com"
```

In the command above the **"id_ed25519"** and **"id_ed25519.pub"** files are stored in the /root/.ssh directory.

```
root@ns7:~ # cd /root/.ssh
root@ns7:~/.ssh # ls
id_ed25519      known_hosts
id_ed25519.pub  known_hosts.old
```

The contents of the file **"id_ed25519.pub"**, which we will place in your Github account.<br><br/>
## 3.  Added SSH keys to Github
GitHub allows you to use SSH keys to authenticate. Make sure you have a Github account and have logged in. Here is how to add SSH keys to Github.

1.  Open the Github website https://github.com
2.  Click your logo at the top right
3.  Click the "Settings" menu
4.  On the left side, click "SSH and GPG keys" with the "key" icon

After that, you follow the image instructions below.

![Github SSH and GPG Keys](https://gitflic.ru/project/iwanse1212/unixwinbsd/blob/raw?file=Github%20SSH%20and%20GPG%20Keys.jpg)

![Github Add New SSH Key](https://gitflic.ru/project/iwanse1212/unixwinbsd/blob/raw?file=Github%20Add%20New%20SSH%20Key.jpg)


## 4. Upload Repository to Github
Now you have connected to the Github server via SSH port 22. Next, you can upload the repository from your local computer to the Github server.

Follow the instructions below to upload the repository to Github.

1.  Open the Github website "https://github.com/"
2.  Select Github account, if you have more than one Github account
3.  Click "Repositories" at the top
4.  Click the "New repository" button
5.  Follow the picture

![crate new repository on github](https://gitflic.ru/project/iwanse1212/unixwinbsd/blob/raw?file=Create%20new%20Repository%20Github.jpg)

Now you have a public repository called "blogmetatag" which we will place in the "/usr/local/etc/" folder. On your local computer, open Putty and open the "/usr/local/etc/blogmetatag" folder.

```
root@ns7:~ # cd /usr/local/etc/blogmetatag
root@ns7:/usr/local/etc/blogmetatag # ls
```

We start by uploading the Repository to the Github server.

```
root@ns7:/usr/local/etc/blogmetatag # echo "# blogmetatag" >> README.md root@ns7:/usr/local/etc/blogmetatag # git init root@ns7:/usr/local/etc/blogmetatag # git add README.md root@ns7:/usr/local/etc/blogmetatag # git commit -m "first commit" root@ns7:/usr/local/etc/blogmetatag # git branch -M main root@ns7:/usr/local/etc/blogmetatag # git remote add origin git@github.com:unixwinbsd/blogmetatag.git root@ns7:/usr/local/etc/blogmetatag # git push -u origin main  Enter passphrase for key '/root/.ssh/id_ed25519': **_Enter the SSH password_** Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Writing objects: 100% (3/3), 225 bytes | 225.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:unixwinbsd/blogmetatag.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'. root@ns7:/usr/local/etc/blogmetatag #
```

".gitignore" is a text file containing a list of files and directories to exclude. The ".gitignore" file is not tracked and is not uploaded, in the git repository. You can use it to stop tracking some hidden files or exclude uploading large files. Github limits file sizes, the maximum size limit per file is 100 MB.
