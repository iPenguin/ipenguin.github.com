---
title: "Static IPv6 Address Assignment using OpenWrt"
permalink: "/2018/03/static-ipv6-address-assigment-using-openwrt.html"
date: "2018-03-17 22:05:12"
updated: "2018-03-17 22:05:12"
excerpt: "Using OpenWrt's LuCI interface to assign the static IPv6 addresses would fail with no explanation, and all of the servers need uniquely assigned DUIDs"
header:
  teaser: "/assets/images/openwrt.png"
categories: [ ipv6, openwrt, networking, tips ]
comments: true
---

## Static IPv6 Address Assignment using OpenWrt

I just spent the better part of a day trying to get static IPv6 addresses assigned to some of my home server devices and I wanted to document what I figured out after piecing together several different sources of information.

### The Setup

I purchased 3 [rock64](https://www.pine64.org/?page_id=7147) [Single Board Computers](https://en.wikipedia.org/wiki/Single-board_computer) to experiment with ansible, freeIPA, docker, kubernetes and anything else I can come up with. Since I am using these devices as servers I want to assign static IPv4 and IPv6 addresses to each one of them.

I want to configure the servers as follows:

    +------------+--------------+--------------+
    | Hostname   | IPv4 Address | IPv6 Suffix  |
    +------------+--------------+--------------+
    | master.lan | 192.168.0.10 |       ::0:10 |
    | node01.lan | 192.168.0.11 |       ::0:11 |
    | node02.lan | 192.168.0.12 |       ::0:12 |
    +------------+--------------+--------------+

OpenWrt has you write out the IPv6 Suffix so the entry in LuCI or `/etc/config/dhcp` for the `master` host looks like this: `00000010`

### The Problem

Creating the static IPv6 entries using OpenWrt's web interface appeared to work, however after restarting the server they were still being assigned dynamic addresses. After some searching I found a thread talking about [assigning the DUID in the dhcp config file](https://forum.openwrt.org/viewtopic.php?pid=310160#p310160).

Using that information configuring the DUID for the master server was easy, and rebooting it came up with the static IP and everything was looking pretty good. That is, until I added the entry for node01.

The second DUID was the same as the first, and it would take over the lease and IP address from the master server. At that point I checked them out and noticed that all three devices had the same DUID.

DHCPv6 address assignment requires a unique DUID for each device you want it to assign an address. So the rock64 devices each need a uniquely generated DUID.

I’m not sure if they all had the same DUID because they were cloned from the same OS image and already had a generated DUID when the image was created or if there was another reason. Whatever the reason I needed to find a way to give all three devices a unique DUID.

Since the default DHCP client software on Ubuntu Xenial (the distro I'm using on the rock64's) is dhclient I decided to stick with solutions for that.

So after some more searching I found that [dhclient can be used to create unique DUIDs](https://superuser.com/a/954133). DHCPv6 settings are stored in `/etc/dhcp/dhclient6.conf` (if the file doesn't already exist -- it didn't in my case – you should create it).

### Connecting the Dots

On your server add or edit the following:

    interface "eth0" {
        send dhcp6.client-id 12:34:56:78:90:12:34:56:78:90;
    }

The ID can be any random unique number. To connect the static IPv6 address you need to add an entry to your OpenWrt server.

While you can edit the IPv6-Suffix from LuCI, you cannot connect that suffix to the server without manually editing the config file on the command line to add an entry for the DUID.

Assuming you already have a statically assigned IPv4 address for the server you can just edit and add 1 line to the /etc/config/dhcp config for the host:

    config host
        option name 'master'
        option dns '1'
        option mac '12:34:56:78:90:AB'
        option ip '192.168.0.10'
        option duid '12345678901234567890'
        option hostid '10'

The DUID entry that you added should had the same ID as the dhcp6.client-id, but you need to remove the colons.

Restart the odhcp daemon to clear any existing leases and to pull in the new configuration:

/etc/init.d/odhcpd restart

You might find you need to delete the leases for the master server before you reboot it so that it gets the new lease from your router, if so, you can delete the corresponding lease file in /var/lib/NetworkManager.

Reboot your server and it should come up with your newly assigned static IPv6 address.

### Final Thoughts

It would make me very happy if LuCI would add the ability to enter the DUID directly into the Web interface, but the fact that it can be done on the command line is good enough for now.

I hope this post was helpful to others.

