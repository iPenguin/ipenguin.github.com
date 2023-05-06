---
title: "Running FreeIPA in Kubernetes"
permalink: "/2023/05/freeipa-in-kubernetes.html"
date: "2023-05-06 20:00:00"
updated: "2023-05-06 20:00:00"
excerpt: "FreeIPA takes a long time to install and the pod was getting killed because it failed the livenessProbe."
header:
  teaser: '/assets/images/k8s.png'
  image: /assets/images/posts/cluster.jpg
  caption: "Photo credit: Brian Milco"
categories: [kubernetes, k8s, development, cluster, freeipa]
comments: true
---

### Background

For the last few months I've been rewriting my Ansible plays and rebuilding my Kubernetes cluster with all the things I've learned since I first built it. In that process I wanted to try again to get a working instance of containerized FreeIPA in Kubernetes.

I've been working on it in my free time and after a couple of months I was able to get a working Kubernetes manifest that will deploy and run.

There were a number of links and posts that were helpful to me in figuring out what I needed to to write the full manifest, and I've linked them below. After my initial push several weeks ago I was able to get a working manifest by setting the `livenessProbe` and `readinessProbe` `initialDelaySeconds` to 1200, However that delay applies to the pod when it is restarted and wont become ready for 20 minutes. So after some research this week I reset the `livenessProbe` and `readinessProbe` and I added a `startupProbe`. I then configured the `initialDelaySeconds` and `periodSeconds` on that `startupProbe` and was able to get a pod that waited for the install to complete on the first start, but restarted in a more timely fashion.

### The YAML

All of the manifests [are available in the repo](https://github.com/iPenguin/bare-metal-kubernetes/tree/master/roles/freeipa/templates). The piece that really fixed most of my issues it right here:

{% highlight yaml %}
  startupProbe:
    httpGet:
      path: /
      port: 80
    # NOTE: if initial installation fails, try increasing this value:
    failureThreshold: 35
    periodSeconds: 30
{% endhighlight %}

This block is saying that it will wait for a successful connection on port 80 of the pod. It's going to make 35 attempts to connect every 30 seconds (35 x 30 = 1050 seconds or 17.5 minutes). Yes it can take FreeIPA more then 15 minutes to install and setup a new instance with the options I've selected. If you find that the installation process is cut off and the container shuts down in the middle of the process try increasing the `failureThreshold`.

### References that were helpful

https://www.linkedin.com/pulse/how-install-freeipa-ubuntu-docker-packopsdev-farshad-nickfetrat/
https://github.com/freeipa/freeipa-container/issues/172
https://github.com/freeipa/freeipa-container/issues/154
https://github.com/freeipa/freeipa-container
https://lists.fedorahosted.org/archives/list/freeipa-users@lists.fedorahosted.org/thread/MHXBWJP7FZPWFRXEDENWQZNOKHJM3HQ6/
