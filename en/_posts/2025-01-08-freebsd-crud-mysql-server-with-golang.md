---
title: FreeBSD14 Crud MysQL Server With Go Lang
date: "2025-01-09 14:11:10 +0100"
id: freebsd-crud-mysql-server-with-golang
lang: en
layout: single
author_profile: true
categories:
  - FreeBSD
tags: "DataBase"
excerpt: CRUD, which stands for Create, Read, Update, and Delete, is a common utility in software development
keywords: go, go lang, freebsd, mysql, crud, golang, app, web
---

Building a Restful application is one of the popular ways to build a cool blog by following a simple and scalable architecture. With CRUD, users can easily process MySQL databases. CRUD, which stands for Create, Read, Update, and Delete, is a common utility in software development. The ability to perform CRUD operations is fundamental to any application development.

Go Lang is a popular programming language known for its efficiency, simplicity, and stability, while MySQL is a relational database management system (RDBMS) developed by Oracle based on the structured query language (SQL). While MySQL is one of the most popular database technologies in the modern big data ecosystem, and is now widely and effectively used, regardless of industry, it is clear that anyone involved with enterprise data or IT should have at least a basic understanding of MySQL.

With MySQL, we can learn the relational database system can immediately build a data storage system that is easy, fast, and secure. Data in MySQL is arranged according to the relational model. In this model, tables consist of rows and columns, and the relationships between data elements all follow a strict logical structure. Often people who are new to learning Golang do not know how to store data in MySQL because there are not many online resources. This article will help you understand the concept of Golang with MySQL.<br><br/>
## 1. System Specifications Used
> OS: FreeBSD 14.
>
> IP Address: 192.168.5.2.
>
> Go version: go-1.19.
>
> MySql Server.
>
> Hostname: ns7.
<br><br/>
## 2. Create a Golang project
The first step is to create a working directory to set up the development environment and start a Golang project. Follow the steps below to set up a new Golang project.

```
root@ns7:~ # cd /var
root@ns7:/var # mkdir -p FreeBSD-Golang-MySQL
root@ns7:/var # cd FreeBSD-Golang-MySQL
root@ns7:/var/FreeBSD-Golang-MySQL #
```

Once you have finished creating the Go language directory, create a new file named "main.go".

```
root@ns7:/var/FreeBSD-Golang-MySQL # touch main.go
root@ns7:/var/FreeBSD-Golang-MySQL # chmod +x main.go
```

In the file "/root/var/FreeBSD-Golang-MySQL/main.go", you enter the script below.

```
package main

import (
	"database/sql"
	"html/template"
	"log"
	"net/http"

	_ "github.com/go-sql-driver/mysql"
)

// Employee Struct
type Employee struct {
	ID   int
	Name string
	City string
}

// Open Connection with MySQL Driver
func dbConnect() (db *sql.DB) {
	dbDriver := "mysql"
	dbUser := "jhondoe"
	dbPass := "router"
	dbName := "goblog"
	db, err := sql.Open(dbDriver, dbUser+":"+dbPass+"@/"+dbName)
	if err != nil {
		panic(err.Error())
	}
	return db
}

// Read All Templates on folder template
var tmpl = template.Must(template.ParseGlob("template/*"))

// Index Page
func Index(w http.ResponseWriter, r *http.Request) {
	db := dbConnect()
	rows, err := db.Query("SELECT * FROM Employee ORDER BY id DESC")
	if err != nil {
		panic(err.Error())
	}
	emp := Employee{}
	res := []Employee{}
	for rows.Next() {
		var id int
		var name, city string
		err = rows.Scan(&id, &name, &city)
		if err != nil {
			panic(err.Error())
		}
		emp.ID = id
		emp.Name = name
		emp.City = city
		res = append(res, emp)
	}
	tmpl.ExecuteTemplate(w, "Index", res)
	defer db.Close()
}

// Show Single Item
func Show(w http.ResponseWriter, r *http.Request) {
	db := dbConnect()
	nID := r.URL.Query().Get("id")
	rows, err := db.Query("SELECT * FROM Employee WHERE id = ?", nID)
	if err != nil {
		panic(err.Error())
	}
	emp := Employee{}
	for rows.Next() {
		var id int
		var name, city string
		err = rows.Scan(&id, &name, &city)
		if err != nil {
			panic(err.Error())
		}
		emp.ID = id
		emp.Name = name
		emp.City = city
	}
	tmpl.ExecuteTemplate(w, "Show", emp)
	defer db.Close()
}

// Show New Page
func New(w http.ResponseWriter, r *http.Request) {
	tmpl.ExecuteTemplate(w, "New", nil)
}

// Edit Item
func Edit(w http.ResponseWriter, r *http.Request) {
	db := dbConnect()
	nID := r.URL.Query().Get("id")
	rows, err := db.Query("SELECT * FROM Employee WHERE id = ?", nID)
	if err != nil {
		panic(err.Error())
	}
	emp := Employee{}
	for rows.Next() {
		var id int
		var name, city string
		err = rows.Scan(&id, &name, &city)
		if err != nil {
			panic(err.Error())
		}
		emp.ID = id
		emp.Name = name
		emp.City = city
	}
	tmpl.ExecuteTemplate(w, "Edit", emp)
	defer db.Close()
}

// Insert Item
func Insert(w http.ResponseWriter, r *http.Request) {
	db := dbConnect()
	if r.Method == "POST" {
		name := r.FormValue("name")
		city := r.FormValue("city")
		insert, err := db.Prepare("INSERT INTO Employee (name, city) VALUES(?, ?)")
		if err != nil {
			panic(err.Error())
		}
		insert.Exec(name, city)
		log.Println("INSERT: Name: " + name + " | City: " + city)
	}
	defer db.Close()
	http.Redirect(w, r, "/", 301)
}

// Update Item
func Update(w http.ResponseWriter, r *http.Request) {
	db := dbConnect()
	if r.Method == "POST" {
		name := r.FormValue("name")
		city := r.FormValue("city")
		id := r.FormValue("uid")
		insert, err := db.Prepare("UPDATE Employee SET name = ?, city = ? WHERE id = ?")
		if err != nil {
			panic(err.Error())
		}
		insert.Exec(name, city, id)
		log.Println("UPDATE: Name: " + name + " | City: " + city)
	}
	defer db.Close()
	http.Redirect(w, r, "/", 301)
}

// Delete Item
func Delete(w http.ResponseWriter, r *http.Request) {
	db := dbConnect()
	emp := r.URL.Query().Get("id")
	delete, err := db.Prepare("DELETE FROM Employee WHERE id = ?")
	if err != nil {
		panic(err.Error())
	}
	delete.Exec(emp)
	log.Println("DELETE")
	defer db.Close()
	http.Redirect(w, r, "/", 301)
}

func main() {
	log.Println("Server started on: http://192.168.5.2:4000")
	// Routes
	http.HandleFunc("/", Index)
	http.HandleFunc("/show", Show)
	http.HandleFunc("/new", New)
	http.HandleFunc("/edit", Edit)
	http.HandleFunc("/insert", Insert)
	http.HandleFunc("/update", Update)
	http.HandleFunc("/delete", Delete)
	// Start server on port 4000
	http.ListenAndServe("192.168.5.2:4000", nil)
}
```

Note the script http://192.168.5.2:4000, this is the IP and Port that Go Lang will use to access the Web browser. Use the git command to initialize your Go Lang project.

```
root@ns7:/var/FreeBSD-Golang-MySQL # go mod init github.com/iwanse1977/FreeBSD-Golang-MySQL
root@ns7:/var/FreeBSD-Golang-MySQL # go mod tidy
root@ns7:/var/FreeBSD-Golang-MySQL # go get github.com/go-sql-driver/mysql
```

In your working project, create a folder "template", and create the files Edit.tmpl, Footer.tmpl, Header.tmpl, Index.tmpl, Menu.tmpl, New.tmpl, Show.tmpl.

```
root@ns7:/var/FreeBSD-Golang-MySQL # mkdir template
root@ns7:/var/FreeBSD-Golang-MySQL # cd template
root@ns7:/var/FreeBSD-Golang-MySQL/template # touch Edit.tmpl Footer.tmpl Header.tmpl Index.tmpl Menu.tmpl New.tmpl Show.tmpl
```

> [Download the script code of the "*.tmpl" file](https://gitflic.ru/project/iwanse1212/freebsd-golang-mysql).

