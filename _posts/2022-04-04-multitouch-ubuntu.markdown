---
layout: post
title: "How to Enable / Configure Multi-Touch Gestures in Ubuntu 20.04 & Higher" 
categories: linux
date : 2022-04-04 20:13:50
---

I have been looking for this feature for a while and, whilst it does not really provide me with the "expose" feature I was originally looking for, it is a good step in the right direction. 

The instructions below are taken from [this page](https://ubuntuhandbook.org/index.php/2021/06/multi-touch-gestures-ubuntu-20-04/?unapproved=3724225&moderation-hash=53471331b0e1fd399db47cbe9cad69cb#comment-3724225)


Add the repository: 
> `sudo add-apt-repository ppa:touchegg/stable`

Install Touche: 
> `sudo apt install touchegg`

Check it is running: 
> `systemctl status touchegg.service`

If not, run `systemctl status touchegg.service && systemctl start touchegg.service`

Install the graphical interface.
> `sudo apt install flatpak`
> 
> `flatpak install https://dl.flathub.org/repo/appstream/com.github.joseexposito.touche.flatpakref`

Run Touche 👍️


---------------------

You might need this down the line: 

Remove Touche:
> `sudo apt remove --autoremove touchegg`

Remove the repository:
> `sudo add-apt-repository --remove ppa:touchegg/stable`

Remove the graphical interface:
> `flatpak uninstall --delete-data com.github.joseexposito.touche`
