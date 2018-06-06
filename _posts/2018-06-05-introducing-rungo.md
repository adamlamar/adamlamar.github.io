---
layout: post
title: Introducing rungo
---

# Background

The first version manager for any programming language that ever I used was [RVM](https://rvm.io). Still a good option for ruby projects,
RVM offers a lot more control than manual path manipulation tricks or compilation suffixes. Plus, you can try out alternative ruby
implementations with relative ease.

One of the more powerful features is the `.rvmrc` and later `.ruby-version` version files. I like version files because they make it
possible to version control the specific runtime requirements for you, other devs, and CI. Particularly with `.ruby-version`, you can use RVM,
[chruby](https://github.com/postmodern/chruby), or any number of other version managers for a consistent experience. A simple feature,
but really, really helpful.

As with anything, there are also some not so nice tradeoffs. My biggest pain points were difficulty running a version-managed in cron
or other minimal environments (problems finding the installed ruby) and installing a system-wide ruby (sudo, file/dir permissions, oh my).
And understandably, some people _really_ don't like [hooking cd](http://batkin.tumblr.com/post/8847990062/on-rvms-cd-script) and the 
various challenges that can cause.

Regardless, my experience with version managers was definitely influenced by RVM's design, and I don't think I'm the only one. When searching for a 
version manager in Go, the seemingly most popular option [gvm](https://github.com/moovweb/gvm) has a similar interface. While each version manager
feature set varies, the same story is repeated by version managers in other languages.

Over time its become clear to me that RVM's approach works well enough for developer-centric workflows, but I was never happy with it on
an application server or in a CI environment. Containers can abstract away some concerns, but come with their own tradeoffs.

So due various challenges running Go at work in CI, I decided to write my own Go version manager.
Here's the rough set of design constraints:

* Minimal configuration and complexity
* Utilize golang's excellent binary releases (no building from source or custom distributions)
* Function correctly in as many environments as possible
* Be written in Go (because nobody wants to write bash)
* Be free of bash/shell altogether

I'm pretty happy with the result.

# Introducing Rungo

Install:

```
brew install adamlamar/rungo/rungo
```

The basic workflow is like using go as normal:

```
$ go version
time="2018-05-15T18:36:39-06:00" level=info msg="Downloading file https://storage.googleapis.com/golang/go1.10.2.darwin-amd64.tar.gz"
time="2018-05-15T18:36:52-06:00" level=info msg="Successfully extracted \"/Users/alamar/.rungo/1.10.2/go1.10.2.darwin-amd64.tar.gz\""
go version go1.10.2 darwin/amd64
```

Most users only need to be aware of two options:

* Write a `.go-version` file in your directory tree to set the desired version (e.g. `echo "1.10.2" > $HOME/.go-version`). This works well for per-project functionality.
* Set the `GO_VERSION` environment variable (e.g. `export GO_VERSION=1.10.2`). This will override any `.go-version` file.

If you don't set either of these, `rungo` will automatically use the latest available version of Go.

Users can also set `RUNGO_VERBOSE=t` for debugging information.

There are no command line options or other environment variables. You don't have to install a given version of Go before use. It doesn't hook `cd` or care about your shell.

Just (optionally) tell `rungo` the version you'd like to run. That's it.

If you want to learn more, check out [the github project](https://github.com/adamlamar/rungo). And be sure to file issues for any problems or use cases not currently
covered. Looking forward to your feedback!
