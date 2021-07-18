---
title: "Tales of a Bare Metal Kubernetes Cluster"
permalink: "/2021/07/bare-metal-kubernetes-cluster.html"
date: "2021-07-15 20:00:00"
updated: "2021-07-15 20:00:00"
excerpt: "After some research I figured out that the problem was likely platform specific. To test the theory I created 3 x86_64 RHEL8 VMs on my desktop..."
header:
  teaser: '/assets/images/k8s.png'
  image: /assets/images/posts/cluster.jpg
  caption: "Photo credit: Brian Milco"
categories: [kubernetes, k8s, development, cluster, lenovo]
comments: true
---

### Background

For the last couple of years I have been running Docker on a Lenovo ThinkCentre M720q for most of my home network services, but I wanted to move to a Kubernetes cluster so when I began studying for the Certified Kubernetes Administrator (CKA) exam I knew I would take time to create a cluster for my home network as well.

I had some constraints that I wanted to keep in mind as I designed the new system. I wanted to be able to keep the M720q as my NFS server as it has a fast 1TB m.2 drive and the 3 [Rock64](https://www.pine64.org/devices/single-board-computers/rock64/) arm SBCs I was planning on using would not have the capacity or speed. I repurposed the Rock64 boards from another project I had worked on and I add the M720q to the cluster. I had had some stability issues with the Rock64 devices before but inspired by products like the [PicoCluster](https://www.picocluster.com/) I thought they should be good enough for this project.

### False Starts

The initial setup went well and I used the [Ubuntu 18.04 container image](https://wiki.pine64.org/index.php/ROCK64_Software_Release#Ubuntu_18.04_Bionic_containers_.28DockerCE_.26_Kubernetes.29_Image_.5BmicroSD_.2F_eMMC_Boot.5D_.5B0.9.14.5D) for the Rock64 devices and the M720q already had RHEL8 installed on it so I left that alone. As I began configuring the cluster and adding storage the stability problems I was having with the Rock64 devices were becoming more annoying. They would freeze and need to be restarted intermittently. I tried several different configurations requiring me to re-install and reconfigure the nodes on several occasions, and I tried several different PSU options as well.

I tied to keep the Rock64 units in the cluster until I spent the better part of a week trying to get past a bug in [Ceph](https://ceph.io/). The bug was:

    stderr: Volume group "ceph-[volume-id]" has insufficient free space ([X] extents): [X+1] required.

After some research I figured out that the problem was likely platform specific. To test the theory I created 3 x86_64 RHEL8 VMs on my desktop and used Ansible to setup the cluster on them. Ceph provisioned the additional drives in the VMs without issue and I was able to get storage up and running and continue on with the deployment.

### Finding My Way

I continued development on the VMs for several weeks before deciding that the Rock64 boards were more trouble then they were worth. I wiped my desktop machine and turned it into a cluster node, and purchased a 3rd ThinkCentre device on eBay. I scrounged up some spare drives and memory and tried to get the specs as close to each other as possible.

Here are the hardware specs for the nodes:

 | Model | M720q | M75q-1 | M920q |
 :--|:--|:--|:--
 | **CPU** | Intel i5-8600T @ 2.3GHz | AMD Ryzen 5 PRO 3400GE @ 2.3GHz | Intel i5-8500T @ 2.1GHz |
 | **Cores** | 6 | 8 | 6 |
 | **RAM** | 32G | 32G | 12G |
 | **Primary Storage** | 1TB m.2 | 250GB m.2 | 128GB m.2 |
 | **Ceph Storage** | 250GB SSD | 256GB SSD | 240GB SSD |

All of the code for this cluster setup can be found in the [bare metal kubernetes](https://github.com/iPenguin/bare-metal-kubernetes) repo on my GitHub account.

### Final Thoughts

As a future upgrade I've considered removing Kubuernetes from the platform and creating 2 VMs on each device. The first VM would have the minimum specs needed to be a master Kubernetes node, and the second VM would take up all additional resources as a worker node. This would allow for the 3 master nodes needed to have a quorum in a production environment and also have 3 worker nodes.

I've been using SBC arm boards for about 6 years, but they've always been less reliable and/or had performance issues. While I still think they have a place I'm tired of fighting the device, platform specific bugs, and reliability issues while I'm trying to learn new software. I've been very impressed with the Lenovo ThinkCentre Tiny form factor machines since I bought my first one a couple of years ago.

I like that this cluster only takes up a little bit more space then the Rock64 cluster would have. I was sorry to lose my desktop machine and to have the higher power usage, but the resulting rock solid reliability and performance were worth it.
