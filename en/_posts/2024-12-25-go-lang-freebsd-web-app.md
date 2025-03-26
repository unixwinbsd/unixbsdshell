---
title: Go Lang and FreeBSD Collaboration - Creating Simple Web Apps
date: "2024-12-25 15:11:10 +0100"
id: go-lang-freebsd-web-app
lang: en
layout: single
author_profile: true
categories:
  - FreeBSD
tags: "WebServer"
excerpt: Google uses Go in a number of its internal systems, including the Kubernetes container orchestration platform and the Google Search search engine.
keywords: go, go lang, freebsd, web, app, apache, static
---
Go or Go lang is a popular programming language widely used in industry to create various applications, including web servers, blogs, networking tools and system utilities. Go is known for its simplicity, efficiency, and concurrency support, making it suitable for building scalable, high-performance systems.

Go is used by many companies and organizations, including:

1.  **Google :**  Google uses Go in a number of its internal systems, including its Kubernetes container orchestration platform and Google Search search engine.
2.  **Netflix :**  Netflix uses Go across a number of its systems, including its data pipeline and recommendation engine.
3.  **Dropbox :**  Dropbox uses Go for a number of its services, including its file sync engine and server infrastructure.
4.  **Uber :**  Uber uses Go in a number of its systems, including distributed data storage and routing engines.

One of Go's greatest strengths lies in its suitability for developing web applications. Google's Go lang offers great performance, is easy to deploy, and has many of the essential tools you need to build and deploy scalable web services in its standard library.

This tutorial will explain and guide you to create a practical example of building a web application with Go and deploying it to the internet network so that it can be read by many people. This guide will cover the basics of using Go's built-in HTTP server and templating language, as well as how to interact with external APIs.

## 1. System Specifications
-   OS: FreeBSD 13.2
-   Hostname/Domain: ns1@unixexplore.com
-   IP Address: 192.168.5.2
-   go version: go1.20.7 freebsd/amd64
-   Port GoLang Web: 8999

## 2. Create Simple Web Applications
The first step to get started with Go is to install it on our server computer. For the Go installation process, you can read our previous article: "[How to Install GOLANG GO Language on FreeBSD](https://www.blogger.com/u/1/blog/post/edit/3047631139734470358/8206179386387319774#)". You can practice all the contents of this article on Linux server machines such as Ubuntu, Debian and others, and can also be applied to Windows and MacOS operating systems.  

As basic material or opening material, we start by creating a simple web application that only displays text in a web browser. To start, we will create a working folder/directory, here are the steps.

```
root@ns1:~ # mkdir -p /var/GoogleBlog
root@ns1:~ # cd /var/GoogleBlog
```

The explanation of the script above is to create a working directory with the name GoogleBlog folder which we place in the /var/GoogleBlog folder. After we have successfully created the working directory, the next step is to write the program code in GO language.  

Go programs are organized into packages. A package is a collection of source files in the same directory that are compiled together. Functions, types, variables, and constants defined in one source file are visible to all other source files in the same package.  

A repository contains one or more modules. A module is a collection of related Go packages released together. A Go repository usually contains only one module, located at the root of the repository. A file named go.mod declares the module path:import path prefix for all packages in the module. The module contains packages in a directory containing the go.mod file as well as subdirectories of that directory, up to the next subdirectory containing other go.mod files (if any).


