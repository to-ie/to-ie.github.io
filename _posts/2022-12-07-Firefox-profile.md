---
layout: post
title: "Firefox Profilemaker " 
categories: tech
date : 2022-12-07 09:00:50
---

[This tool](https://ffprofile.com/#start) will help you to create a Firefox profile with the defaults you like.

You select which features you want to enable and disable and in the end you get a download link for a zip-file with your profile template. You can for example disable some functions, which send data to Mozilla and Google, or disable several annoying Firefox functions like Mozilla Hello or the Pocket integration.

Each Setting has a short explanation and for the non obvious settings links to resources describing the feature and the possible problems with it. 

## Installing the new profile
Installing

- Optional: add a new profile to keep the old one
    - Run `firefox -no-remote -ProfileManager`
    - Create a new profile
- Type about:support into the url bar.
- Press the open profile folder button.
- Quit Firefox.
- Delete everything from the new profile (you will lose all existing data from the profile).
- Unzip the `profile.zip` archive into the folder.
- If Existent: Unzip the `enterprise_policy.zip` archive to Firefox installation directory.
- Start Firefox again. If you made a new profile, you can use it with `firefox -no-remote -P profilename`.
- Open the add-on manager and update the extensions.
