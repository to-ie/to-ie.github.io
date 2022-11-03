---
title: "The volume “filesystem root” has only X bytes disk space remaining" 
description : "Annoying repetitive error message... It was time to fix" 
tags : [ "technology", "linux"]
date : 2022-04-04T19:54:50
author : "Theodore" 
---

I recently switched my main OS from Windows to Ubuntu Mate and, I must admit, it has been quite painless! (much to my surprise)

But recently, I started getting this error message: 
> The volume “filesystem root” has only 600 bytes disk space remaining

![File System Root Error](/img/blog-posts/file-system-root-error.png)

I did try to ignore it for a while until I was looking to install Ms Teams (don't ask) and I simply did not have enough space. 🤦‍♂️

So it was time... Time to get to the bottom of it. 

That's when I stumbled upon [this](https://itectec.com/ubuntu/ubuntu-the-volume-filesystem-root-has-only-0-bytes-disk-space-remaining/) article. 

I discovered `df`, which shows you your "Disk Free" Space. 

I learnt this: 

```
Don't delete files without first knowing what they are, of course. But, in general, you won't break your system if you delete files in the following directories:

    /tmp (user temp data -- these are commonly all deleted every reboot anyway)
    /var/tmp (print spools, and other system temporary data)
    /var/cache/* (this one can be dangerous, research first!)
    /root (the root user's home directory)

In addition to the locations above, the following locations are common culprits:

    /opt (many third-party apps install here, and don't clean up after themselves)
    /var/log (log files can eat up a lot of space if there are repetitive errors)
```

And finally, I Googled `Resize Ubuntu Partition`. 💡 Turns out partitions need to be next to one another in Gparted in order to extend one into the other. 

And... Tadaaaaa! All fixed now!