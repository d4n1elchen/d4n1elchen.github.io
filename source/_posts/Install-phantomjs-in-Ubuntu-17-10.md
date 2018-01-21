---
title: Install phantomjs in Ubuntu 17.10
date: 2018-01-21 13:45:29
tags: ['phantomjs', 'front-end', 'web']
categories: 'Web'
---

Phantom is a headless webkit for front-end testing.

You can download executable file on their [website](http://phantomjs.org/download.html). ([Direct link for phantomjs-2.1.3](https://github.com/ariya/phantomjs/releases/download/2.1.3/phantomjs))

But you will soon get following error if you're using Ubuntu 17.10.

```
phantomjs: error while loading shared libraries: libicui18n.so.55: cannot open shared object file: No such file or directory
```

It seems like we have missing dependency of `libicu55`. If you try to resolve this dependancy issue by using apt, you won't be successful.

```bash
$ sudo apt install libicu55
Reading package lists... Done
Building dependency tree
Reading state information... Done
Package libicu55 is not available, but is referred to by another package.
This may mean that the package is missing, has been obsoleted, or
is only available from another source

E: Package 'libicu55' has no installation candidate
```

You have to download the package manually and install it.

```
$ wget http://security.ubuntu.com/ubuntu/pool/main/i/icu/libicu55_55.1-7ubuntu0.3_amd64.deb -P /tmp/
$ sudo dpkg -i /tmp/libicu55_55.1-7ubuntu0.3_amd64.deb
```

Testing phantomjs

```
$ vim hello.js
```
```
console.log('Hello, world!');
phantom.exit();
```

```
$ ./phantomjs hello.js
```
```
Hello, world!
```
