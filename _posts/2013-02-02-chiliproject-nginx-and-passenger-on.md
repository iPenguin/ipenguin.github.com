---
title: "ChiliProject, Nginx and Passenger on a Raspberry Pi"
layout: "post"
permalink: "/2013/02/chiliproject-nginx-and-passenger-on.html"
uuid: "8010456291845471441"
guid: "tag:blogger.com,1999:blog-3270817893928434685.post-8010456291845471441"
date: "2013-02-02 03:44:00"
updated: "2013-02-10 08:15:31"
description: 
blogger:
    siteid: "3270817893928434685"
    postid: "8010456291845471441"
    comments: "0"
categories: [Raspberry Pi, ChiliProject, Passenger, Nginx]
author: 
    name: "Brian C. Milco"
    url: "http://www.blogger.com/profile/05356031750889872461?rel=author"
    image: "http://img2.blogblog.com/img/b16-rounded.gif"
comments: true
---

### Background

Yes, I bought a [Raspberry Pi](http://downloads.element14.com/raspberryPi1.html). :-) 
I have some additional things I like to try out on it, but first I wanted to set it up as a low power NAS and web server.
One of the projects I wanted to put on the server was the Chiliproject which I use for some person projects, 
but I ran into some issues that I'm going to outline in this post so that you can get the ChiliProject up and running on your Raspberry Pi.  
  
The biggest drawback for nginx is that it doesn't have a plug-in architecture like Apache, you have to compile in any additional modules you want to use. 
Unfortunately the deb packages on Raspian don't come with support for Passenger built-in. 

As a side note I did try to use thin to serve the ChiliProjcet, but let's just say I didn't have any luck with that.

### Some notes before your start

I used [Raspbian Server Edition](http://sirlagz.net/tag/raspbian-server-edition/) for this setup. 
When I tried the same process on a [Raspbmc](http://www.raspbmc.com/) card I had lying around it wouldn't compile nginx.

I chose to break out the apt-get install lines so you could see what lines were requiring the additional packages you need to install. 
If you don't care feel free to combine them into one line and come back when it's done, as some of the later steps will pull in quite a few extra packages.

The Pi isn't the fastest device so you'll probably want to set aside an evening to get this up and running. 
Also the very first time you try to connect to the setup it's pretty slow to respond, but subsequent, connections and page loads go at a reasonably usable speed.

### Installing what you need

If you have nginx and passenger installed already go ahead and remove them.

{% highlight bash %}
sudo apt-get remove nginx ruby-passenger
{% endhighlight %}

I'm using a copy of the original /etc/init.d/nginx to launch nginx at startup. I had to edit the script to point to the /opt/nginx/sbin/nginx binary.

{% highlight bash %}
sudo apt-get install rubygems

sudo gem install passenger

sudo apt-get install libcurl4-openssl-dev libssl-dev

sudo passenger-install-nginx-module
{% endhighlight %}

Option 1 works nicely, but if you want IPv6 you'll have to select option 2, download nginx source, follow the instructions, and add the flag --with-ipv6 when prompted for additional flags.

Note: At one point the passenger-install-nginx-module script told me I was missing the ruby1.8-dev package when in reality I was missing  the ruby1.9.x-dev files, hopefully this will save someone several hours of tinkering.

The the bare minimum nginx configuration you need to add to the http section:

{% highlight bash %}
passenger_root /var/lib/gems/1.8/gems/passenger-3.0.19;
passenger_ruby /usr/bin/ruby1.8;
passenger_pool_idle_time 604800; # 1 week in seconds

server {
    listen      80;
    server_name  chiliproject.lan;

    root /srv/www/public;
    passenger_enabled on;

    #keep one process running at all times so you don't have
    #to reload the ChiliProject every time you connect.
    passenger_min_instances 1; 
                               
}
{% endhighlight %}

You can add this fragment directly into the /opt/nginx/conf/nginx.conf file, inside the http section, or you can add an include statement, and put the server {} section in another file.

NOTE: /srv/www/ is the toplevel folder for the chiliproject files.

{% highlight bash %}
sudo apt-get install libmysqlclient-dev libpq-dev libmagickcore-dev libmagickwand-dev libsqlite3-dev
{% endhighlight %}

This will install a number of extra packages.

{% highlight bash %}
cd /srv/www/
{% endhighlight %}

I'm assuming you've already [downloaded ChiliProject](https://www.chiliproject.org/projects/chiliproject/wiki/Download) and extracted it to it's final location, in this case I'm using /srv/www/

{% highlight bash %}
sudo apt-get install bundler

sudo bundle install [--without test development]
{% endhighlight %}

### Finishing up

The basics are now installed and setup. You can use [the official guide](https://www.chiliproject.org/projects/chiliproject/wiki/Installation) to do the rest of the installation starting at step 3. 
Everything else went smoothly for a clean install, and for an upgrade from an old Redmine install.

I hope this helps you get your own setup running!

EDITED: Added additional configuration information for Passenger to keep the application running to speed up subsequent connections. 
Thanks to [Felix Schäfer](https://www.chiliproject.org/boards/1/topics/2371?r=2373#message-2373) for the suggestion.


