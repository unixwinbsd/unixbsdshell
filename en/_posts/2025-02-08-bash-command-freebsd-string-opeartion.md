---
title: Bash Command in FreeBSD - Advanced String Operations - Building Custom Functions
date: "2025-02-08 11:17:10 +0300"
id: bash-command-freebsd-string-opeartion
lang: en
layout: single
author_profile: true
categories:
  - FreeBSD
tags: "UnixShell"
excerpt: Whether you are a beginner or an experienced shell scripter with Bash commands, mastering scripting, creating and utilizing Bash functions is
keywords: bash, command, cli, shell, freebsd, string, integer
---

Let’s talk about the power of Bash command functions in the form of shell scripting. Bash functions are like your constant companions that empower you to create reusable blocks of code that can be called at multiple points in your script or even in other scripts. Imagine the convenience! With Bash functions, you can simplify your scripts, make them easier to understand, and reduce the dreaded code duplication.

Whether you are a beginner or an experienced shell scripter with Bash commands, mastering scripting, creating and utilizing Bash functions is an absolute game changer in your journey as a developer. So, buckle up and embrace the power of Bash functions as you take your scripting skills to the next level!.

In this article we’ve written on blogger, we’ll dive into the world of Bash scripting and explore how to create custom string functions to perform advanced string operations. We cover important functions like ltrim, rtrim, trim, reverse, and len, and provide code examples to help you build these functions into your Bash scripts. By the end of this article, you’ll have a better understanding of how to create custom functions to save time and streamline your Bash scripting workflow.

![Bash Command on FreeBSD 14](https://gitflic.ru/project/iwanse1212/unixwinbsd/blob/raw?file=Bash%20Command%20on%20FreeBSD%2014.jpg)

## 1. What is Bash Scripting?
Bash scripting is a method of automating tasks that utilizes the Bash shell, which serves as the default command-line interface for various Unix-based operating systems such as FreeBSD, Linux and macOS. Bash scripts are programs coded in the Bash scripting language, which can be run in a CMD terminal or as stand-alone script files.

Bash scripting makes it easy to accomplish a variety of tasks, from running simple commands to performing complex automation and system administration functions. With Bash scripting, you can automate repetitive tasks, develop system maintenance scripts, build deployment workflows, and more. Additionally, Bash scripts can be used to manage software systems, install and configure applications, and manipulate data. So there’s a lot you can get with Bash.

## 2. What is a Bash Function?
Bash functions can be defined as a sequence of commands defined in a Bash script or specified interactively on a CMD command, depending on the use case. Once defined, Bash functions can be called multiple times in a script or other scripts, just like regular shell commands to shorten the time required and reduce coding complexity.

Bash functions allow you to create reusable blocks of code that can perform complex operations, organize code, and simplify scripts. You can pass arguments to Bash functions, which you can then use within the function to perform operations on those arguments. Additionally, Bash functions can return values ??that can be used in other parts of your script.

There are many types of Bash script functions, such as declaration, call, return, argument, and others.

In this article, we'll show you how to create some commonly used custom string functions when working with Bash, such as ltrim, rtrim, trim, reverse, and len. The following are Bash custom string functions commonly used by application developers.

### a. LTRIM
So, you know how sometimes you have a string with some annoying leading whitespace characters that you just can’t get rid of? Well, that’s where ltrim comes in. This function is like a magic eraser that removes all the unwanted whitespace from the beginning of your string.

Below is an example script for the ltrim function.

```
function ltrim {
    echo "${1#"${1%%[![:space:]]*}"}"
}
```

### b. RTRIM
Examples in the RTRIM package may be handy for your own analyses. RTRIM package is a new version of the program TRIM, which was rebuilt as an R package for the freely available and widely used open-source statistical program R.

Here is the code for the rtrim function:

```
function rtrim {
    echo "${1%"${1##*[![:space:]]}"}"
}
```

### c. TRIM
This function is like the love child of the ltrim and rtrim functions we talked about earlier. It takes your input string and removes both leading and trailing whitespace characters, leaving you with a clean and tidy string that’s ready to use.

Below is an example script for the ltrim function.

```
function trim {
    echo "$(rtrim "$(ltrim "$1")")"
}
```

### d. REVERSE
The reverse function is like a magician, taking your input string and flipping it so that the characters are in reverse order. It’s like looking at your reflection in a mirror, but for strings!

The reverse function uses a loop to iterate over each character in your input string, building a new string in reverse order as it goes. The loop is like a little assembly line, producing a new string that is the reverse of the original string.

Here’s an example script for the reverse function:

```
function reverse {
  local str="$1"
  local reversed=""
  local len=${#str}
  for ((i=$len-1; i>=0; i--))
  do
    reversed="$reversed${str:$i:1}"
  done
  echo "$reversed"
}
```

### e. LEN
The len function acts as a measuring tape for your string and can help you figure out how long it is. With len, you can easily find out how many characters are in your string, which can be very useful for all sorts of string operations.

The len function uses a built-in Bash feature that returns the length of a variable. This feature is like a magic wand, waving it over your string and telling you exactly how many characters are in there.

Here is a sample code for the len function:

```
function len {
    echo "${#1}"
}
```

### f. UPPERCASE
This function takes an input string and converts it to all uppercase characters. It’s like a big, bold statement saying “Hey, look at me, I’m using ALL CAPS!”

The uppercase function uses a Bash built-in command called “tr”. This command is like an interpreter, taking your input string and replacing all lowercase characters with their uppercase equivalents.

Here’s the code for the uppercase function:

```
function uppercase {
    echo "$1" | tr '[:lower:]' '[:upper:]'
}
```

### g. LOWERCASE
This function is used to take an input string and convert it all to lowercase characters.

Here is the code for the lowercase function:

```
function lowercase {
    echo "$1" | tr '[:upper:]' '[:lower:]'
}
```

### h. SUBSTITUTE
This function can change all your code to whatever you want. It’s like having a magic wand that can change your strings to whatever you want!

The substitution function uses a Bash built-in command called “sed”. This command is like a surgeon that can perform complex substitutions on your input strings, based on the search string and the replacement string you provide.

Here is the code for the substitution function:

```
function substitute {
    echo "$1" | sed "s/$2/$3/g"
}
```

### i. TRUNCATE
The truncate function truncates a string to a specified length and appends an ellipsis ("…") at the end. Here’s the code for the truncate function:

```
function truncate {
    local str="$1"
    local len="$2"
    if [ "${#str}" -gt "$len" ]; then
        echo "${str:0:$len}..."
    else
        echo "$str"
    fi
}
```

### j. COUNT
The count function is used to count the code you type. The count function uses a built-in Bash feature that counts how many times a pattern appears in a string. This feature is like a calculator that can add up all the occurrences of your search string in your input string.

Here is an example code for the count function:

```
function count {
    echo "$1" | awk -v FS="$2" '{print NF-1}'
}
```

### k. SPLIT
The split function is like a knife that can cut your input string into pieces, based on a specified delimiter. The split function uses a Bash built-in command called "read". This command is like a waiter that serves your input string in small, manageable pieces, based on a specified delimiter.

Below is an example code for the split function:

```
function split {
    local IFS="$2"
    read -ra arr <<< "$1"
    echo "${arr[@]}"
}
```

### l. CAPITALIZE
This function is like a stylist that can give your input string a brand new look by capitalizing the first letter of each word. It’s like putting on a fancy hat or a snazzy pair of shoes - it just makes your string look more polished and put together!

So, how does it work? Well, the capitalize function uses a Bash built-in command called “sed”. This command is like a fashion designer that can make subtle changes to your input string, based on a pattern that you provide.

Below is an example code for the split function:

```
function capitalize {
    echo "$1" | sed 's/\b\([a-z]\)/\u\1/g'
}
```

Bash functions are essential for creating modular and reusable scripts. They allow you to group related commands together and encapsulate complex logic to make your code easier to read and maintain.

In this article, I’ve covered the basics of Bash functions, including how to define and call functions and the most popular Bash function scripts. I also covered some common use cases for Bash functions to better get you going on your Bash use journey. By using Bash functions in your scripts, you can write cleaner, more modular code that is easier to understand and maintain!.
