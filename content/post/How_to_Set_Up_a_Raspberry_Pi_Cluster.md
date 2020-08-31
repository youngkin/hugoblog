---
title: "How to Set Up a Raspberry Pi Cluster"
description: "This article tells you what you need to buy, the OS install, and the networking setup to create a Raspberry Pi cluster."
image: "/images/router.jpeg"
date: 2020-08-26T13:01:02-06:00
tags: ["Raspberry Pi", "Networking", "Cluster"]
categories: ["Raspberry Pi"]
---

> Photo by [Jonathan](https://unsplash.com/@isodme) on [Unsplash](https://unsplash.com/s/photos/sailor?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)


## The full text of this article can be found on [Medium](https://medium.com/better-programming/how-to-set-up-a-raspberry-pi-cluster-ff484a1c6be9?source=friends_link&sk=c679e5274ed39a5002e2b9ac97c966c7).

---

This article covers what you need to know to setup your own Raspberry Pi cluster, including you what you need to buy, how to install the OS and configure the Raspberry Pis, how to set up networking, and other things you need to know to sinplify ongoing administration and maintenance of the cluster.

This is the second article of the series described in [Develop and Deploy Kubernetes Applications on a Raspberry Pi Cluster](https://medium.com/better-programming/develop-and-deploy-kubernetes-applications-on-a-raspberry-pi-cluster-fbd4d97a904c?source=friends_link&sk=df18f8cdfc8b90aa25b2b6676346d1ec). After completing the steps outlined in this article, you'll be ready to learn [How to Install Kubernetes on a Raspberry Pi Cluster](https://medium.com/better-programming/how-to-install-kubernetes-on-a-raspberry-pi-cluster-49ad9a762d08?source=friends_link&sk=d9a6bba68c42a1321dd9008a92a1a330), the next article in the series.
As this article references a couple of other articles relevant to the task, you might find it helpful to have them open in another browser window so you can cross-reference between them and this one as needed.
There are five main sections to this article:

1. My Target Cluster Topology - This section describes what we'll build in this article.
2. Initial Setup - This section describes what equipment is needed, how to install the OS, and how to complete the initial configuration of each Raspberry Pi.
3. Configure the Network - This section describes the process to follow to configure the actual Raspberry Pi cluster. At the end of the section, we'll have a working cluster.
4. Set Up Keyless SSH Access Between Hosts- This section will cover how to configure key-less ssh access between the nodes of the cluster. It will also describe how to set up a reverse-ssh tunnel to a host in the external network to allow any permitted host in the external network ssh access into any of the permitting cluster nodes. This isn't possible out of the box because the Pi router won't forward IP traffic from the external network into the cluster network.
5. Set Up i2cssh- This section covers how to have simultaneous terminal sessions open to several hosts. i2cssh is a terminal multiplexer similar to tmux. With i2cssh, you can have commands replicated to every terminal session. This is useful if you want to execute the same commands on multiple hosts at the same time.

## Articles in the Series

The full set of articles are:

* [Set up a Raspberry Pi Cluster](https://medium.com/better-programming/setup-a-raspberry-pi-cluster-ff484a1c6be9) : Describes how to set up and configure a group of Raspberry Pis into a cluster. This includes defining the networking within the cluster and to an external network.
* [Install Kubernetes on a Raspberry Pi Cluster](https://medium.com/better-programming/install-kubernetes-on-a-raspberry-pi-cluster-49ad9a762d08): This includes installing & configuring Kubernetes as well as building and deploying a simple service. This latter step involves a minimal application that can be used to test the Kubernetes cluster as well as simple external network access to the application.
* [Install Kubernetes Ingress on a Raspberry Pi Cluster](https://medium.com/@RichYoungkin/install-kubernetes-ingress-on-a-raspberry-pi-cluster-e8d5086c5009): Ingress is a useful way to support external access to services inside the cluster. In this article I’ll be describe how to install, configure, and test Traefik as an Ingress Controller. This article also includes installing and configuring Helm. Helm will be used to deploy Traefik. Helm will also be used in later articles in the series.
* [Kubernetes Application Monitoring on a Raspberry Pi Cluster](https://medium.com/better-programming/kubernetes-application-monitoring-on-a-raspberry-pi-cluster-fa8f2762b00c) — Install Prometheus, Grafana, and an EFK (Elasticsearch/Fluentd/Kibana) as a monitoring stack. It also covers installing Telegraf on the cluster to monitor machine level metrics for hosts on the cluster. E.g., CPU load and temperature, disk usage
* [How to Install MariaDB on a Raspberry Pi](https://medium.com/better-programming/how-to-install-mysql-on-a-raspberry-pi-ad3f69b4a094) — The title says it all. One thing to note, MariaDB is a drop-in replacement for MySQL