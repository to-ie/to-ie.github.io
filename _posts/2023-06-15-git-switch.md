---
layout: post
title: "GitSwitch" 
categories: systems
date : 2023-06-15 09:34:50
---

I have 2 GitHub accounts, one for work and one for pleasure. But I only use the one laptop. So, switching from one account to another, in the terminal has always been a pain in the back. So I thought I'd work on finding a technical solution to this technical problem. And I built... [Gitswitch](https://github.com/to-ie/gitswitch)!

## What is it? 
This script helps with users who have multiple GitHub profiles and require to switch between them on a regular basis. It currently works on Ubuntu (and other Linux distros) and supports 2 GitHub accounts only. 

## Some assumptions
I assume that you already have GitHub CLI installed (if not, check [this page](https://cli.github.com/manual/installation) out) and that you are using [Personal Access Tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) to connect to your GitHub account.


## Some prep
Before installing Gitswitch, you might want to have the following info ready, for each account:

- A nickname for the account (ie: 'work')
- The GitHub username
- The email address linked to the account
- The personal access key for the GitHub account (generate one here)


## How it works
The script reads from the file `gscurrent`. If the content is `profile1`, it will replace the gitconfig files (`.gitconfig` and `.git-credentials`) with the config files of `profile2` (and vice versa).

It's not pretty, but it does the job. 👍

## Installation and configuration
To install Gitswitch, follow the instructions below. 

Let's keep things clean. Head over to your Desktop:
```
cd ~/Desktop
```

Clone this repository:
```
git clone https://github.com/to-ie/gitswitch
```

and launch the setup:
```
bash ~/Desktop/gitswitch/switch.sh
```

Follow the configuration instructions provided by the script. 
<br>Note: you will need some basic information about your accounts (check the pre-requisit section above).

Once you have installed and configured Gitswitch, let's clean up after ourselves:
```
rm -r ~/Desktop/gitswitch
```

## Switching profile
And finally, to switch profiles, simply open a terminal and type `gitswitch`. Follow the prompts and you're good to go!


## Gitswitch on GitHub
Check out the repo at [https://github.com/to-ie/gitswitch/](https://github.com/to-ie/gitswitch/)