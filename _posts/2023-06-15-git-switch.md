---
layout: post
title: "GitSwitch" 
categories: systems
date : 2023-06-15 11:34:50
---

I have 2 GitHub accounts, one for work and one for pleasure. But I only use the one laptop. So, switching from one account to another, in the terminal has always been a pain in the back. 

Until I wrote this: 

```
#!/bin/bash
# This script switches github profiles
# Made by toie on 20230615
# Works on Ubuntu

GREEN='\033[0;32m' 
NC='\033[0m' # No Color

# Require user prompt to begin script
echo "You are about to switch git profile."
echo "You are currently logged in as"
cat ~/.gitcurrent
echo

read -p "Press Enter to switch profiles" </dev/tty

if grep -q <profile-1> ~/.gitcurrent; then
  cp ~/Documents/git-switcher/gitcredential/<profile-2>/.gitconfig ~/.gitconfig 
  cp ~/Documents/git-switcher/gitcredential/<profile-2>/.git-credentials ~/.git-credentials 
  echo <profile-2> > ~/.gitcurrent
  printf "${GREEN}You are now logged in as <profile-2>${NC}\n"
  echo
  exit
fi

if grep -q <profile-2> ~/.gitcurrent; then
  cp ~/Documents/git-switcher/gitcredential/<profile-1>/.gitconfig ~/.gitconfig 
  cp ~/Documents/git-switcher/gitcredential/<profile-1>/.git-credentials ~/.git-credentials 
  echo <profile-1> > ~/.gitcurrent
  printf "${GREEN}you are now logged in as <profile-1>${NC}\n"
  echo
  exit
fi
```

## How it works

The script reads from the file `.gitcurrent`. If the content is `<profile-1>`, it will replace the gitconfig files (`.gitconfig` and `.git-credentials`) with the config files of `<profile-2>` (and vice versa).

It's not pretty, but it does the job. 👍

## Configuration

The script's files are stored in a directory called `git-switcher` structured as below:

```
├── gitcredential
│            ├── <profile-1>
│            │           ├── .gitconfig
│            │           └── .git-credentials
│            └── <profile-2>
│                ├── .gitconfig
│                └── .git-credentials
└── switch.sh

```

In the folders `<profile-1>` and `<profile-2>`, we store the Github config files for each account. 

We also need to create a file called `.gitcurrent` in your root directory, with the name of either of your profiles stored inside it. 

## Alias setting

I set up an alias in my `.bashrc` file, to help call the script from anywhere: 

```
alias gitswitch="bash <path-to-switch.sh>
```

## Switching profile
And finally, to switch profiles, simply open a terminal and type `gitswitch`. Follow the prompts and you're good to go!

