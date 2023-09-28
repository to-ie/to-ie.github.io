---
layout: post
title: "Adjusting the telemetry settings of Firefox" 
categories: privacy
date : 2023-09-28 09:14:50
---

Have you thought about your browser recently? You know, that program that allows you to access the internet? These days, you basically have the choice between a chromium-based browser (Chrome, Edge, Brave,...) or a non chromium-based one (Safare & Firefox). 

There is a huge portion out there that have gone nuclear on trying to encourage the contiued use of Firefox, as to keep a non Google alternative in the market. And, you know what? I can't fault the thought process. 

In short, the pro-Firefox propaganda would tell you that it's faster, better for privacy, more customisable, open-source,... Simply better! 

As I've been wearing my tinfoil hat on for a few years now, when it comes to browsing the web, I have opted out for a combination of [Firefox](https://www.mozilla.org/en-US/firefox/new/) and the [uBlock Origin plugin](https://addons.mozilla.org/en-US/firefox/addon/ublock-origin/). 

## What is telemetry

Wikipedia says: "Telemetry is the in situ collection of measurements or other data at remote points and their automatic transmission to receiving equipment (telecommunication) for monitoring." VOM. 

Basically, it is the data that is sent back to Mozilla whilst you use their browser (when you use the browser, how you use it, etc...). The idea is that it helps them improve the product, but paranoid people like myself would argue that it's better they don't know anything about me.  

## How to disable telemetry on Firefox

When you first install Firefox, the telemetry settings are switched on, by default. To switch them off, simply follow the steps below:
- Click on the 3 lines on the top right of your window. 
- Click on `Settings`.
- On the left side, click on `Privacy & Security`.
- Scroll down to `Firefox Data Collection and Use`.
- Un-tick the boxes there. 

That's it, that's a first step to disabling telemetry on Firefox. Behold though, there is more! 

There are some additional settings, hidden under the hood, that you can disable too. In your browser url bar, type `about:config`. Then set the following to `false`:

```
toolkit.telemetry.archive.enabled
toolkit.telemetry.bhrPing.enabled
toolkit.telemetry.firstShutdownPing.enabled
toolkit.telemetry.newProfilePing.enabled
toolkit.telemetry.shutdownPingSender.enabled
toolkit.telemetry.unified
toolkit.telemetry.updatePing.enabled
browser.ping-centre.telemetry
browser.newtabpage.activity-stream.telemetry
browser.newtabpage.activity-stream.feeds.telemetry
```

That's it, you have now disabled telemetry on Firefox. 

## Small Bonus
Here is a nifty video on how to configure and use uBlock Origin to protect your online privacy and security. 

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/2lisQQmWQkY?si=i3BzuvdTxvZGWQf4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>