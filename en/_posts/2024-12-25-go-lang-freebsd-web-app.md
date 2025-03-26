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

To compile and run a simple program, first select the module path (we will use the working directory above and create a go.mod file declaring it.

```
root@ns1:~ # cd /var/GoogleBlog
root@ns1:/var/GoogleBlog # go mod init GoogleBlog
go: creating new go.mod: module GoogleBlog
root@ns1:/var/GoogleBlog #
```

The script above will create a go.mod file that we will use to compile and run Go programs. After the go.mod compilation file has been successfully created, the next step is to create the main Go file which we call "main.go" and enter the script code below in the "main.go" file.

```
root@ns1:/var/GoogleBlog # ee main.go

package main

import (
	"net/http"
	"os"
)

func indexHandler(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("<h1>Good Morning, FreeBSD!</h1>"))
}

func main() {
	port := os.Getenv("PORT")
	if port == "" {
		port = "8999"
	}

	mux := http.NewServeMux()

	mux.HandleFunc("/", indexHandler)
	http.ListenAndServe("192.168.5.2:"+port, mux)
}
```

Then we run the "main.go" file by typing in the script below.

```
root@ns1:/var/GoogleBlog # go run main.go
```

We can see the results by opening the Mozilla Firefox or Google Chrome web browser, in the address bar menu type "http://192.168.5.2:8999/" and the results will look like the image below.

gambar selamat pagi

Another example of a script that you can practice, delete all the scripts in the "main.go" file and replace them with the script below.

```
package main
import (
    "fmt"
    "net/http"
)
type msg string
func (m msg) ServeHTTP(resp http.ResponseWriter, req *http.Request) {
   fmt.Fprint(resp, m) 
}
func main() {
    msgHandler := msg("Hello from Web Server in Go")
    fmt.Println("Server is listening...")
    http.ListenAndServe("192.168.5.2:8999", msgHandler)
}
```

Run the Go application.

```
root@ns1:/var/GoogleBlog # go run main.go
```

Open Google Chrome and type "http://192.168.5.2:8999/" in the address bar menu and you can see the results in Google Chrome.

## 3. Determining Routes and Handler Functions in Go
There are many ways to perform HTTP path routing in Go, including the http.ServeMux standard library, but they only support basic prefix matching. Apart from that, there are also many third-party router libraries. In Go, routing can be done in several ways, including:

1.  By utilizing the http.HandleFunc().
2.  function Implementing the http.Handler interface in a struct, to then be used in the http.Handle() function.
3.  Create your own multiplexer using the http.ServeMux struct.
4.  And others

We can define paths for applications and corresponding controller functions. Routers in Go are a combination of HTTP methods (such as GET , PUT , or POST ) and URL patterns that Go applications must respond to. To determine routing, we will use the http.HandleFunc function which takes a pattern and a handler function as arguments. For example.

```
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
  // Add handler code here
})
```

The above script will determine the path that responds to the GET request at the URL "/". The controller function will receive two arguments, the ResponseWriter interface, which is used to write the response to the client, and the Request structure, which contains information about the request made by the client.

Next, we have to write a controller function for the application path we are going to create. These functions will be tasked with carrying out appropriate actions for each path and returning a response to the client. For example, if we are creating a server that displays a list of users, you might have a controller function that queries the database for the list of users and then renders an HTML template with the list of users that has been stored in the database.

To render an HTML template, you must first parse the template file using the template.ParseFiles function. Like the example below.

```
tmpl, err := template.ParseFiles("template.html")
if err != nil {
  http.Error(w, err.Error(), http.StatusInternalServerError)
  return
}
```

HTML templates are files that contain the layout and content of application pages. We can include placeholders for dynamic data that will be passed by the controller function. For example, here's a simple HTML template that displays a list of users. Type the script below in the file.

```
root@ns1:~ # cd /var/GoogleBlog
root@ns1:/var/GoogleBlog # ee index.html

<html>
<head>
        <title>{{.Title}}</title>
</head>
<body>
        <h1>{{.Title}}</h1>
        <ul>
                {{range .Users}}
                        <li>{{.Name}} ({{.Age}} years old)</li>
                {{end}}
        </ul>
</body>
<html>
<head>
        <title>{{.Title}}</title>
</head>
<body>
        <h1>{{.Title}}</h1>
        <ul>
                {{range .Users}}
                        <li>{{.Name}} ({{.Age}} years old)</li>
                {{end}}
        </ul>
        <form action="/add" method="post">
                <label>Name: <input type="text" name="name"></label>
                <br>
                <label>Age: <input type="number" name="age"></label>
                <br>
                <input type="submit" value="Add user">
        </form>
</body>
</html>
```

After that, we create the main Go file which we name "/var/GoogleBlog./main.go" and enter the script below in the "main.go" file.

```
root@ns1:/var/GoogleBlog # ee main.go

package main

import (
	"html/template"
	"net/http"
	"strconv"
)

type User struct {
	Name string
	Age  int
}

var users []User

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		// Parse the HTML template
		tmpl, err := template.ParseFiles("index.html")
		if err != nil {
			http.Error(w, err.Error(), http.StatusInternalServerError)
			return
		}

		// Define the data for the template
		data := struct {
			Title string
			Users []User
		}{
			Title: "Users",
			Users: users,
		}

		// Render the template with the data
		err = tmpl.Execute(w, data)
		if err != nil {
			http.Error(w, err.Error(), http.StatusInternalServerError)
			return
		}
	})

	http.HandleFunc("/add", func(w http.ResponseWriter, r *http.Request) {
		if r.Method == http.MethodPost {
			// Parse the form values
			name := r.FormValue("name")
			ageStr := r.FormValue("age")
			age, err := strconv.Atoi(ageStr)
			if err != nil {
				http.Error(w, "Invalid age", http.StatusBadRequest)
				return
			}

			// Add the user to the list of users
			users = append(users, User{Name: name, Age: age})

			// Redirect the user back to the homepage
			http.Redirect(w, r, "/", http.StatusSeeOther)
		}
	})

	fs := http.FileServer(http.Dir("static"))
	http.Handle("/static/", http.StripPrefix("/static/", fs))

	http.ListenAndServe("192.168.5.2:8999", nil)
}
```

The next step is, we run the Go server, by typing the command below.

```
root@ns1:/var/GoogleBlog # go run main.go
```

We can see the results by opening the Google Chrome or Yandex web browser, in the address bar menu type "http://192.168.5.2:8999/", then it will look like the image below.

gambar 2

In this tutorial, we learn how to create a web application in Go from scratch. we also learn how to import the necessary packages, define controller paths and functions, create an HTML template, and start an HTTP server. We have finished this article for now, we will continue to the next article with material that is quite difficult and a challenging script.
