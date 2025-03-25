---
title: Ruby Installation Guide on OpenBSD with Ruby $HOME PATH Path
date: "2025-02-11 17:11:10 +0100"
id: ruby-installation-openbsd-path-home
lang: en
layout: single
author_profile: true
categories:
  - FreeBSD
tags: "DataBase"
excerpt: Ruby on Rails is a web framework. There are several Ruby maintainers available to install Ruby.
keywords: openbsd, unix, ruby, rails, path, home, environment
---


Ruby is one of the most widely used and easy to use programming languages. Ruby is an open-source object-oriented interpreter that can be installed on Linux systems. Many programmers prefer Python over Ruby to start learning basic programming, but Ruby can handle large web frameworks and web applications.

Once you start learning Ruby, you will find it less machine-like and less repetitive. If you are confused about the difference between Ruby and Ruby on Rails, I must mention that they are not the same. Ruby is an open-source, object-oriented, and general-purpose programming language that is one of the most popular programming languages.

Whereas Ruby on Rails is a web framework. There are several Ruby maintainers available to install Ruby. Ruby maintainers allow the use of multiple versions and help switch between Ruby versions. The most commonly used Ruby maintainers are rbenv and rvm. Ruby is also available in the OpenBSD repository.<br><br/>
## 1. Computer Specifications Used
> OS: OpenBSD 7.6 amd64
> 
> Host: Acer Aspire M1800
> 
> Uptime: 15 mins
> 
> Packages: 111 (pkg_info)
> 
> Shell: ksh v5.2.14 99/07/13.2
> 
> Terminal: /dev/ttyp0
> 
> CPU: Intel Core 2 Duo E8400 (2) @ 3.000GHz
> 
> Memory: 59MiB / 1775MiB
> 
> Versi Ruby: ruby 3.3.5 (2024-09-03 revision ef084cc8f4) [x86_64-openbsd]
> 
> Versi Gem: # 3.5.16
> 
> Versi Bundler: 2.6.5

## 2. How to Install Ruby on OpenBSD
Ruby is written in the C programming language, and its syntax is similar to Perl and Python. Installing Ruby on a Linux system does not require much computing skills. An active internet connection and root privileges are required to install the Ruby language on any system. If you are a beginner to Ruby, this post will help you. Here, I will guide you through installing and getting started with Ruby on Debian, Red Hat, and Arch-based Linux systems.

```
ns2# pkg_add ruby
```

Once you have successfully installed Ruby, proceed to create a symlink. Follow the guide below to create a Ruby symlink.

```
ns2# ln -sf /usr/local/bin/ruby33 /usr/local/bin/ruby
ns2# ln -sf /usr/local/bin/bundle33 /usr/local/bin/bundle
ns2# ln -sf /usr/local/bin/bundler33 /usr/local/bin/bundler
ns2# ln -sf /usr/local/bin/erb33 /usr/local/bin/erb
ns2# ln -sf /usr/local/bin/gem33 /usr/local/bin/gem
ns2# ln -sf /usr/local/bin/irb33 /usr/local/bin/irb
ns2# ln -sf /usr/local/bin/racc33 /usr/local/bin/racc
ns2# ln -sf /usr/local/bin/rake33 /usr/local/bin/rake
ns2# ln -sf /usr/local/bin/rbs33 /usr/local/bin/rbs
ns2# ln -sf /usr/local/bin/rdbg33 /usr/local/bin/rdbg
ns2# ln -sf /usr/local/bin/rdoc33 /usr/local/bin/rdoc
ns2# ln -sf /usr/local/bin/ri33 /usr/local/bin/ri
ns2# ln -sf /usr/local/bin/syntax_suggest33 /usr/local/bin/syntax_suggest
ns2# ln -sf /usr/local/bin/typeprof33 /usr/local/bin/typeprof
```

Don't forget to create a PATH path for the gem.

```
ns2# echo "PATH=$PATH:$HOME/.local/share/gem/ruby/3.3/bin; export PATH" >> /.kshrc
```
<br><br/>
## 3. Check Ruby Version
This step is to ensure that Ruby is properly installed on your OpenBSD system.

```
ns2# ruby -v
ruby 3.3.5 (2024-09-03 revision ef084cc8f4) [x86_64-openbsd]
ns2# gem -v
3.5.16
ns2# bundler -v
Bundler version 2.6.5
```

Once all versions are displayed correctly, you can also test Ruby by creating a simple application, like the following example.

```
ns2# mkdir -p /usr/local/etc/APPRuby
ns2# cd /usr/local/etc/APPRuby
```

In the "/usr/local/etc/APPRuby" folder, you create a file called "testruby.rb".

```
ns2# touch testruby.rb
```

After that, in the file "/usr/local/etc/APPRuby/testruby.rb", you type the script below.

```
puts "Hello, Ruby!. OpenBSD"
```

The final step is to run Ruby.

```
ns2# ruby testruby.rb
Hello, Ruby!. OpenBSD
```

Now that you know how to create basic Ruby programs, you can continue exploring the language and its capabilities.
