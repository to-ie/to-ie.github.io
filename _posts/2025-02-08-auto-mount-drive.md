---
layout: post
title: "Automount drives on UUbuntu server" 
categories: self-hosted
date : 2025-01-18 09:14:50
---

When I set up my home server, I noticed an issue—my drives weren’t mounting automatically. After some research, I discovered several ways to fix this, but I wanted a solution that wasn’t tied to specific drives and could scale easily as my storage needs grew. In this post, I’ll share the method that worked best for me, allowing any drive to be mounted seamlessly without manual intervention.

Introducing: usbmount, mount any USB drive automatically when plugged in.

## Step 1: Download and Install `usbmount`


Download the source files:

```
sudo apt update

# If you don't have git yet
sudo apt install git 

git clone https://github.com/rbrito/usbmount.git

cd usbmount
```
In order to build this package, debhelper and build-essential are required. The package can be built with the simple commands:

```
# Install dependencies
sudo apt-get update && sudo apt-get install -y debhelper build-essential

# Build
sudo dpkg-buildpackage -us -uc -b
```

Finally, install the package:

```
sudo dpkg -i usbmount_*.deb || sudo apt-get -f install
```

## Step 2: Enable Automount

Modify /etc/usbmount/usbmount.conf if needed:

```
sudo nano /etc/usbmount/usbmount.conf
```

Change the line below to include all popular formats:
```
FILESYSTEMS="vfat ext2 ext3 ext4 hfsplus exfat ntfs"
```

That is all. 