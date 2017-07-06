---
title: "ext Filesystems and the Missing 5%"
permalink: "/2011/12/ext-filesystems-and-missing-5.html"
uuid: "3997253902596511394"
guid: "tag:blogger.com,1999:blog-3270817893928434685.post-3997253902596511394"
date: "2011-12-14 01:34:00"
updated: "2012-11-25 00:40:10"
excerpt: "turn off the reserve 5% on ext filesystems"
blogger:
    siteid: "3270817893928434685"
    postid: "3997253902596511394"
    comments: "0"
header:
    teaser: '/assets/images/linux-sm.jpg'
categories: [ext, filesystems, space]
comments: true
---

First why all the fuss about 5%? Because 5% of 2TB = 100GB.

When formatting a partition under Linux 5% of the capacity is automatically reserved for use by the system in case you run out of space and the operating system needs to write to log files, or preform privileged tasks. However if you're creating this new partition as your home partition or you just want an additional data drive it doesn't make sense to discard 5% of the space before you start using it, however there is an option to reclaim that space.

{% highlight bash %}
$ sudo tune2fs -m 0 /dev/sdb1 # <-- your partition may vary!
{% endhighlight %}

-m Sets the percentage to be used by the system, so you can set it to whatever you want.

I usually end up forgetting this command and have to look it up so this blog is as much a reminder to myself as anything else.
