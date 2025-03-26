---
title: Ruby on Rails Installation and Configuration Guide on OpenBSD 7.6
date: "2025-02-12 15:15:19 +0100"
id: ruby-rails-openbsd-instal-configuration
lang: en
layout: single
author_profile: true
categories:
  - OpenBSD
tags: "WebServer"
excerpt: Rails is a framework for building websites. As such, Rails establishes conventions for easier collaboration and maintenance
keywords: ruby, rails, gem, bundle, openbsd, unix, bundler, rake
---

Ruby is a programming language created 20 years ago by Yukihiro “Matz” Matsumoto. Based on its popularity and usage, the Ruby programming language is ranked in the top ten. Ruby's popularity is supported by the Rails application which is already very well-known in creating static websites.

While Rails is a software library that extends the Ruby programming language. Rails creator David Heinemeier Hansson gave it the name "Ruby on Rails," although it is often simply called "Rails." Rails is software code added to the Ruby programming language. Technically, it is a package library (specifically, RubyGem), which is installed using a command-line interface on Unix or Linux operating systems.

Rails is a framework for building websites. As such, Rails establishes conventions for easier collaboration and maintenance. These conventions are codified as the Rails API (application programming interface, or directives that control code). The Rails API is documented online and explained in books, articles, and blog posts. Learning Rails means learning how to use Rails conventions and its API.

In this article we will explain the process of installing and configuring Rails on the OpenBSD 7.6 operating system.

## 1. How to Install Ruby on Rails on OpenBSD
To run Rail on OpenBSD, you must first enable Ruby by installing it. Below are the commands to install Ruby on OpenBSD.

```
ns3# pkg_add update
ns3# pkg_add upgrade
ns3# pkg_add ruby
```

After the Ruby installation process is complete, you continue by checking Ruby. This is to ensure whether Ruby has been installed or not on OpenBSD.

```
ns3# ruby -v
ruby 3.3.5 (2024-09-03 revision ef084cc8f4) [x86_64-openbsd]
ns3# bundler -v
Bundler version 2.6.6
```
For more details about the Ruby installation process on OpenBSD, you can read the [previous article](https://penaadventure.com/en/freebsd/2025/02/11/ruby-installation-openbsd-path-home/).<br><br/>
# 2. Create a Symlink file
Similar to FreeBSD, binary files in OpenBSD are located in the /usr/local/bin directory. All binary files from programs you have installed are stored in this directory.

This directory is also where the Rails binary files are stored, but before we create a symlink file, run the command below to install Rails.

```
ns3# gem install --user-install rails
```

Once you have finished installing Rails, proceed to check the installed Rails version.

```
ns3# rails -v
Rails 8.0.2
```

Then we continue by creating a Symlink with the command below.

```
ns3# ln -sf /usr/local/bin/rails33 /usr/local/bin/rails
```

## 3. Creating Apps with Rails
Rails comes with a number of scripts, called generators. Rails scripts are designed to make the developer's job easier by creating everything needed to create a particular application. One of them is the new application generator, which provides the foundation of a Rails application, so you don't have to write it again.

To create an application or Blog with Rails, you must first determine the directory to store all the application files.

Below are the Rail commands used to create a new application.

```
ns3# rails new blogsite
```

### a. Update Rails
Then proceed to update Rails.

```
ns3# cd blogsite
ns3# bundle install
ns3# bundle update
```

### b. Install Nokogiri
To complete your Rails, also install nokogiri.

```
ns3# bundle install --path=~/.gem
ns3# gem install --user-install nokogiri
```

### c. Run Rail
This section is the most important part of the article. In this section we will run Rails. If all the configurations above have been done well, without any scripts missed, now is the time to run Rails with the command below.

```
ns3# rails s -b 192.168.5.3
```

To see the results, open the Google Chrome web browser and type "http://192.168.5.3:3000/". If the configuration is correct, the "Rails" logo will appear on your monitor screen.

Sampai disini anda telah menyelesaikan konfigurasi OpenBSD dan Ruby on Rails dan anda siap untuk membuat berbagai aplikasi dengan Rails.
