---
title: FreeBSD14 Installation Ruby Gemfile and Bundle With Path Environemnt
date: "2024-12-28 14:10:10 +0200"
id: starship-cross-prompt-freebsd
lang: en
layout: single
author_profile: true
categories:
  - FreeBSD
tags: "WebServer"
excerpt: One of Ruby's advantages is that it has a web development framework known as Ruby on Rails or often abbreviated as Rails
keywords: ruby, freebsd, gemfile, bundle, bundler, rails
---

For beginner programmers, perhaps many are still not familiar with Ruby because it is rarely used as a basis. Ruby is one of the superior programming languages for developing website applications. The Ruby programming language was designed with a focus on simplicity and productivity. Its intuitive and English-like syntax makes it easy to understand and use, especially for programming beginners.

One of Ruby's advantages is that it has a web development framework known as Ruby on Rails or often abbreviated as Rails. Before Rails, developers often spent a lot of time writing code over and over again. However, with Rails, this process is faster because Rails has the CoC (Convention over Configuration) principle and the DRY (Don't Repeat Yourself) principle.

CoC is an approach where the system provides built-in conventions to simplify common tasks, reducing the need for developers to configure every detail. While DRY encourages reducing duplication in code. This principle emphasizes the importance of keeping information or logic in one single place in the code, ensuring the code is more maintainable and minimizing errors.

On FreeBSD, managing and running a Ruby application usually involves determining the Ruby version and dependencies that will form the library to reference our project. Ruby has lots of dependencies, and you have to install all of them according to the project you are working on. Look at the image below.

> Ruby is a dynamic object-oriented programming language. Ruby was designed with a focus on simplicity and productivity.

In this article, we will learn about the Ruby installation process and file library dependencies for Ruby. To complete the contents of this article, we will also explain the process of creating a Ruby PATH. We have implemented the entire contents of this article on the FreeBSD 13.3 server, and you can also apply it to FreeBSD 14.<br><br/>
## 1. Install Ruby From ports and PKG
There are two ways to install applications on FreeBSD, namely the ports system and the PKG package. In this article, we will use both methods. Before you start installing Ruby, update and install Ruby dependencies. Follow the method below.
