---
title: "git status in your bash prompt"
layout: "post"
permalink: "/2011/10/git-status-in-your-bash-prompt.html"
uuid: "4278004634738619082"
guid: "tag:blogger.com,1999:blog-3270817893928434685.post-4278004634738619082"
date: "2011-10-06 17:47:00"
updated: "2011-10-06 17:47:04"
thumbnail: "http://1.bp.blogspot.com/-ByhzGgPIxYE/To1ARLGbsZI/AAAAAAAAACg/1NXKG_-kbSg/s72-c/prompt1.png"
description: 
blogger:
    siteid: "3270817893928434685"
    postid: "4278004634738619082"
    comments: "0"
categories: [awk, bash, PS1, git, prompt]
author: 
    name: "Brian C. Milco"
    url: "http://www.blogger.com/profile/05356031750889872461?rel=author"
    image: "http://img2.blogblog.com/img/b16-rounded.gif"
comments: true
---

I love git, but I found myself typing 'git status' all the time. 
The problem with that is that you can get anything from 2 lines to multiple screens of output depending on what you're doing. 

After seeing [this blog](http://majewsky.wordpress.com/2011/09/13/a-clear-sign-of-madness/) I got the itch to put a little git info into my custom prompt. 
After several weeks of looking around at other solutions and tinkering on my own with an awk script I've come up with this: 

One note, because this script pulls in so much information from git it can take some time to return a prompt for large repositories or across the network.

![prompt1](/images/posts/1.png)
My standard prompt

It starts with a hash (#) to make it a comment if it gets copy/pasted, host name in green, path in blue.


![prompt2](/images/posts/2.png)
A clean repository on the master branch

![prompt3](/images/posts/3.png)
A repository with some changes

The changes (in order from left to right): The local branch contains 1 commit not on the remote branch. 
There is 1 staged change (green), 2 unstaged changed (yellow), no unmerged changes (magenta), 4 untracked files (red), and 1 stashed change (in yellow on the right).

The code for my PS1 and the awk script are below.

{% gist iPenguin/1266869 %}