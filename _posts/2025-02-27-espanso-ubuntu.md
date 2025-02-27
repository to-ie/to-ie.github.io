---
layout: post
title: "Installing Espanso on Ubuntu 24.04 (Wayland)" 
categories: Ubuntu
date : 2025-02-27 09:14:50
---

I spent too much time trying to figure this one out. So I thought I'd write it up for the next person in need. 

## The issue
Installing Espanso based on [the documentation](https://espanso.org/docs/install/linux/#choosing-the-right-install-method) has always been quite straightforward until now. But, since Ubuntu 24.04, some users are hit with a depency issue that is impossible to solve: 

The errors vary: 
```
dpkg: dependency problems prevent configuration of espanso-wayland:
 espanso-wayland depends on libwxgtk3.0-gtk3-0v5; however:
  Package libwxgtk3.0-gtk3-0v5 is not installed.
 espanso-wayland depends on libwxgtk3.0-gtk3-0v5 (>= 3.0.4+dfsg); however:
  Package libwxgtk3.0-gtk3-0v5 is not installed.
 espanso-wayland depends on libwxbase3.0-0v5 (>= 3.0.4+dfsg); however:
  Package libwxbase3.0-0v5 is not installed.

dpkg: error processing package espanso-wayland (--install):
 dependency problems - leaving unconfigured
Errors were encountered while processing:
 espanso-wayland
```

or 

```
espanso: error while loading shared libraries: libwx_gtk3u_html-3.0.so.0: cannot open shared object file: No such file or directory
```

And the worst part is that `libwxgtk3.0-gtk3-dev` is nowhere to be found!

## Attempts to fix it
I tried the snap installation (got the same dependency issue), tried the [compilation from source](https://espanso.org/docs/install/linux/#wayland-compile), I got a whole other range of issues.


## How I did it
First and foremost, I can't take credit for any of this. OP is [davidvv](https://github.com/davidvv) and solution is [here](https://github.com/espanso/espanso/issues/1793#issuecomment-1919055522).

Reminder, I am using Wayland (and the below is only useful if you are too).

### Download the deb package
I downloaded the deb version 2.2.1 of the tool:
```
wget https://github.com/espanso/espanso/releases/download/v2.2.1/espanso-debian-wayland-amd64.deb
```

### Download and install the dependencies
Before installing it, let's install the (God foresaken) dependencies. To do so, you need to  add the source deb http://cz.archive.ubuntu.com/ubuntu focal main universe to your Ubuntu system:
```
echo "deb http://cz.archive.ubuntu.com/ubuntu focal main universe" | sudo tee -a /etc/apt/sources.list
```
then 
```
sudo apt update
```
and finally install libwxgtk3.0
```
sudo apt install libwxgtk3.0-gtk3-dev
```

### Install Espanso
Final steps! Install Espanso:
```
dpkg -i espanso-debian-wayland-amd64.deb 
```

Register as a service:
```
espanso service register
```
And run!
```
espanso start
```

Voilà!