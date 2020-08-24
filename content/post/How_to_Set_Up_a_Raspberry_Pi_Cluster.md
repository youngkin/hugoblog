---
title: "How to Set Up a Raspberry Pi Cluster"
description: "Everything you need to buy, install, and stand up your own datacenter"
featured_image: "/images/router.jpeg"
date: 2020-08-23T22:01:07-06:00
tags: ["Raspberry Pi"]
categories: []
---

[![Router with cables](/images/router.jpeg "Jonathon @ Unsplash")](https://unsplash.com/@isodme)

## The full text of this article can be found on [Medium](https://medium.com/better-programming/how-to-set-up-a-raspberry-pi-cluster-ff484a1c6be9?source=friends_link&sk=c679e5274ed39a5002e2b9ac97c966c7).

This is the second article of the series described in [Develop and Deploy Kubernetes Applications on a Raspberry Pi Cluster](https://medium.com/better-programming/develop-and-deploy-kubernetes-applications-on-a-raspberry-pi-cluster-fbd4d97a904c?source=friends_link&sk=df18f8cdfc8b90aa25b2b6676346d1ec). After completing the steps outlined in this article, you'll be ready to learn [How to Install Kubernetes on a Raspberry Pi Cluster](https://medium.com/better-programming/develop-and-deploy-kubernetes-applications-on-a-raspberry-pi-cluster-fbd4d97a904c?source=friends_link&sk=df18f8cdfc8b90aa25b2b6676346d1ec), the next article in the series.
As this article references a couple of other articles relevant to the task, you might find it helpful to have them open in another browser window so you can cross-reference between them and this one as needed.
There are five main sections to this article:
1. My Target Cluster Topology - This section describes what we'll build in this article.
2. Initial Setup - This section describes what equipment is needed, how to install the OS, and how to complete the initial configuration of each Raspberry Pi.
3. Configure the Network - This section describes the process to follow to configure the actual Raspberry Pi cluster. At the end of the section, we'll have a working cluster.
4. Set Up Keyless SSH Access Between Hosts- This section will cover how to configure key-less ssh access between the nodes of the cluster. It will also describe how to set up a reverse-ssh tunnel to a host in the external network to allow any permitted host in the external network ssh access into any of the permitting cluster nodes. This isn't possible out of the box because the Pi router won't forward IP traffic from the external network into the cluster network.
5. Set Up i2cssh- This section covers how to have simultaneous terminal sessions open to several hosts. i2cssh is a terminal multiplexer similar to tmux. With i2cssh, you can have commands replicated to every terminal session. This is useful if you want to execute the same commands on multiple hosts at the same time.
