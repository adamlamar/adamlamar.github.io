---
layout: post
title: rungo on Windows
---

[rungo](https://github.com/adamlamar/rungo) is a simple version manager for Go, similar in functionality to
[gvm](https://github.com/moovweb/gvm) / [RVM](https://rvm.io/), but completely different in design.

Since rungo is 100% implemented in Go, not bash or any shell-like language, it is much more portable across
different operating systems.

Recently I was able to improve the packaging and usage of rungo in Windows such that it provides a similar experience
to a unix-based OS.

The installation process is pretty straightforward:

* [Download the latest release](https://github.com/adamlamar/rungo/releases/latest) for your OS and architecture
* Extract the release zip file
* Copy the extracted binaries (`go.exe`, `gofmt.exe`, and `godoc.exe`) to somewhere in your PATH (e.g. `C:\Windows`)

Once installed, Windows users can fully utilize rungo as it was intended:

```
C:\Users\user>echo 1.9.6 > .go-version

C:\Users\user>go version
time="2018-06-10T21:52:14-06:00" level=info msg="Downloading sha256 file https://storage.googleapis.com/golang/go1.9.6.windows-amd64.zip.sha256"
time="2018-06-10T21:52:14-06:00" level=info msg="Downloading file https://storage.googleapis.com/golang/go1.9.6.windows-amd64.zip"
time="2018-06-10T21:54:29-06:00" level=info msg="Extracting \"C:\\\\Users\\\\user\\\\AppData\\\\Local\\\\rungo\\\\1.9.6\\\\go1.9.6.windows-amd64.zip\""
go version go1.9.6 windows/amd64
```

Or

```
C:\Users\user>set GO_VERSION=1.10.2

C:\Users\user>go version
time="2018-06-10T21:12:03-06:00" level=info msg="Downloading sha256 file https://storage.googleapis.com/golang/go1.10.2.windows-amd64.zip.sha256"
time="2018-06-10T21:12:03-06:00" level=info msg="Downloading file https://storage.googleapis.com/golang/go1.10.2.windows-amd64.zip"
time="2018-06-10T21:13:39-06:00" level=info msg="Extracting \"C:\\\\Users\\\\user\\\\AppData\\\\Local\\\\rungo\\\\1.10.2\\\\go1.10.2.windows-amd64.zip\""
go version go1.10.2 windows/amd64
```

If you're using powershell, set the environment variable using `$env`:

```
PS C:\Users\user> $env:GO_VERSION="1.10.2"
PS C:\Users\user> go version
time="2018-06-10T21:12:03-06:00" level=info msg="Downloading sha256 file https://storage.googleapis.com/golang/go1.10.2.windows-amd64.zip.sha256"
time="2018-06-10T21:12:03-06:00" level=info msg="Downloading file https://storage.googleapis.com/golang/go1.10.2.windows-amd64.zip"
time="2018-06-10T21:13:39-06:00" level=info msg="Extracting \"C:\\\\Users\\\\user\\\\AppData\\\\Local\\\\rungo\\\\1.10.2\\\\go1.10.2.windows-amd64.zip\""
go version go1.10.2 windows/amd64
```

There you have it! rungo is a Go(lang) version manager with full Windows support.

When you try it out, please [file any issues on github](https://github.com/adamlamar/rungo) or [tweet](https://twitter.com/adam__lamar) feedback.
