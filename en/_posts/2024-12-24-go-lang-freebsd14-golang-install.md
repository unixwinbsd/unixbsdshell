---
title: How to Install GOLANG GO Language on FreeBSD 14
date: "2024-12-24 12:15:10 +0200"
id: go-lang-freebsd14-golang-install
lang: en
layout: single
author_profile: true
categories:
  - FreeBSD
tags: "DataBase"
excerpt: Technological advances make things more practical, including database management.
keywords: freebsd, go, golang, language, unix, freebsd14
---
Go is an Open Source programming language (BSD license), Go was created with the aim of making it easier to create applications that are efficient and easy to distribute. So, Go has binaries for Windows, Mac OS, and of course, for Linux. One of the main features of Go is the efficient management of computer resources. To the point that many claim to be a natural replacement for the C programming language.

On the other hand, Go is a compiled language and has the advantage of reducing latency to a minimum to guarantee optimal performance. Therefore, there are many reasons why program developers make Go their main language. In this article we try to discuss how to install GOLANG or Go Language on the FreeBSD 13.2 system.

To install GO on FreeBSD, you can use the pkg package or FreeBSD ports. The following is how to install GO on [FreeBSD14](https://www.freebsd.org/releases/14.0R/relnotes/).

```
root@ns2:~ # pkg install curl
root@ns2:~ # pkg install go
```

The command above is how to install GO with the pkg package. Below is an example of installing GO with the Ports system.

```
root@ns2:~ # cd /usr/ports/ftp/curl/ && make install clean
root@ns2:~ # cd /usr/ports/lang/go
root@ns1:/usr/ports/lang/go # make config-recursive
root@ns1:/usr/ports/lang/go # make install clean
```

After the installation process is complete, check the GO version.

```
root@ns1:~ # go version
go version go1.21.13 freebsd/amd64
```

In the command above, the Go application used is version go1.20.3 freebsd/amd64. You can also use the env command to view go details.

```
root@ns1:~ # go env
GO111MODULE=""
GOARCH="amd64"
GOBIN=""
GOCACHE="/root/.cache/go-build"
GOENV="/root/.config/go/env"
GOEXE=""
GOEXPERIMENT=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="freebsd"
GOINSECURE=""
GOMODCACHE="/root/go/pkg/mod"
GONOPROXY=""
GONOSUMDB=""
GOOS="freebsd"
GOPATH="/root/go"
GOPRIVATE=""
GOPROXY="https://proxy.golang.org,direct"
GOROOT="/usr/local/go120"
GOSUMDB="sum.golang.org"
GOTMPDIR=""
GOTOOLDIR="/usr/local/go120/pkg/tool/freebsd_amd64"
GOVCS=""
GOVERSION="go1.20.3"
GCCGO="gccgo"
GOAMD64="v1"
AR="ar"
CC="cc"
CXX="clang++"
CGO_ENABLED="1"
GOMOD="/dev/null"
GOWORK=""
CGO_CFLAGS="-O2 -g"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-O2 -g"
CGO_FFLAGS="-O2 -g"
CGO_LDFLAGS="-O2 -g"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -Wl,--no-gc-sections -fmessage-length=0 -fdebug-prefix-map=/tmp/go-build2339347747=/tmp/go-build -gno-record-gcc-switches"
```

The next step, create an environment variable called GOPATH (which will be the location for the installed packages). Enter the following script into the /root/.profile folder.

```
root@ns1:~ # ee /root/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/work/
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
```
<br><br/>
## Testing Go
After go has been successfully installed and configured, it's time to do a go test. In this test we will create a program that will display the words "Hello from freebsd" and "hello, world" to check whether Go is running. Create a file for test go with the following script.

```
root@ns1:~ # cd /usr/local/go120/test
root@ns1:/usr/local/go120/test # ee test.go
package main

import (
    "fmt"
    "runtime"
)

func main() {
    fmt.Println("Hello from", runtime.GOOS)
}
```

Now do a test.

```
root@ns1:/usr/local/go120/test # go run test.go
Hello from freebsd
```

The text "Hello from freebsd" appears, meaning the Go program is running. Now we do the second test.

```
root@ns1:/usr/local/go120/test # ee hello.go
package main
 import "fmt"
 func main() {
 fmt.Printf("hello, world\n")
 }
```

Still active in the /usr/local/go120/test folder, do a test on the "hello.go" file.

```
root@ns1:/usr/local/go120/test # go run hello.go
hello, world
```

There are many programming languages but there is always room for innovation and that is what Go gives us. A very practical and efficient language and with support from giant developers like Google.
