---
layout: post
title: "GitHub backup" 
categories: systems
date : 2023-06-15 08:14:50 
---

As a sysadmin, I regularly backup and test infrastructure, for when the day we need them come. 

One of the items that I backup is GitHub. Granted, there is some argument in the decentralised nature of the platform that ensures that local copies are made on all endpoints, but it will come in handy to have all repositories neatly organised in one place, ready to be re-rolled out (think of GitHub servers going down or your account being compromised and your repositories maliciously deleted).

There are many ways to backup GitHub, but the one below seems to be the simplest. It comes in particularly handy when you need to backup entire GitHub organisations. 

Jose Diaz-Gonzalez from GitHub open source community has developed a tool which allows us to backup different kind of repositories easily. [source](https://leimao.github.io/blog/Backup-GitHub/)

## Install
```
pip install github-backup
```

## Backing up
```
github-backup <org-name-goes-here> --token <personal-access-token> --private --repositories --output-directory <path-to-directory>
```

or, with the blanks filled in:
```
github-backup to-ie --token <personal-access-token> --private --repositories --output-directory ~/Documents/Github-backup/
```

## Access token
For the above command, you will need to [generate a personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#about-personal-access-tokens) on GitHub. 

As `github-backup` is a third-party tool, it might be a good idea to generate a new token each time you do a backup (to avoid it leaking to malicious actors). 👍