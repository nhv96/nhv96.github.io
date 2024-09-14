---
layout: post
title:  "How to install Go in Ubuntu 22.04"
date:   2024-09-14 17:59:30 +0700
category: how-to
---
Go or Golang is a C like syntax, statically typed, compiled programming language. It was developed by the engineers of Google: Robert Griesemer, Rob Pike, and Ken Thompson.

To install Golang in Ubuntu, simply follow these steps:

Head to the Go's project official site [https://go.dev/dl] and download the desired Go binary release that match your machine's architecture.

Remove any previous Go installation by deleting `/usr/local/go`, and extract the download archive into `/usr/local/`.

```
$ sudo rm -rf /usr/local/go && tar -C /usr/local -xzf go1.23.1.linux-<arch>.tar.gz
```

Add `/usr/local/go/bin` to the PATH environment variable.
```
$ export PATH=$PATH:/usr/local/go/bin
```

It is recommended to add the above line to your profile file at `$HOME/.profile`.

**Note**: you should restart your machine in order to apply the changes in profile file.

Verify that you have successfully installed Go.
```
$ go version
go version go1.23.1 linux/amd64
```

[https://go.dev/dl]: https://go.dev/dl