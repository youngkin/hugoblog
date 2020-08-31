---
title: "Install Kubernetes Ingress on a Raspberry Pi Cluster"
description: "This article describes how to setup Kubernetes Ingress using the Traefik ingress controller."
date: 2020-08-26T13:05:37-06:00
draft: false
image: "images/IngressDark.jpeg"
tags: ["Raspberry Pi", "Docker", "Kubernetes", "Ingress", "Traefik"]
categories: ["Raspberry Pi", "Kubernetes"]
---

> Photo by [Tianshu Liu](https://unsplash.com/@tianshu?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/sailor?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

## The full text of this article can be found on [Medium](https://medium.com/better-programming/install-kubernetes-ingress-on-a-raspberry-pi-cluster-e8d5086c5009?source=friends_link&sk=148c7e4d0c276bad20fa7f2ce902736e)

---

This is the fourth article in the series described in [Develop and Deploy Kubernetes Applications on a Raspberry Pi Cluster](https://medium.com/better-programming/develop-and-deploy-kubernetes-applications-on-a-raspberry-pi-cluster-fbd4d97a904c?source=friends_link&sk=df18f8cdfc8b90aa25b2b6676346d1ec). The previous article described [how to install Kubernetes on a Raspberry Pi cluster](https://medium.com/better-programming/how-to-install-kubernetes-on-a-raspberry-pi-cluster-49ad9a762d08?source=friends_link&sk=d9a6bba68c42a1321dd9008a92a1a330). This article is about Kubernetes Ingress. The next article, [Kubernetes Application Monitoring on a Raspberry Pi Cluster](https://medium.com/better-programming/kubernetes-application-monitoring-on-a-raspberry-pi-cluster-fa8f2762b00c?source=friends_link&sk=81553ee8e61a841e43143a8e73c1bb9e), covers setting up Prometheus, Grafana, and a Elasticsearch, Fluentd, and Kibana (EFK) stack on the cluster.

## Overview

Ingress is a mechanism that allows clients in an external network to invoke services running in our Kubernetes cluster. There are other ways of accomplishing this in Kubernetes — e.g., kubectl expose…— NodePorts and load balancers. They each have pros and cons, but suffice it to say that Kubernetes Ingress provides additional capabilities that make it more attractive. These include:

* Service discovery
* Canary-release routing
* Rate limiting
* Circuit breaking
* And much more …
  
## Articles in the Series

The full set of articles are:

* [Set up a Raspberry Pi Cluster](https://medium.com/better-programming/setup-a-raspberry-pi-cluster-ff484a1c6be9) : Describes how to set up and configure a group of Raspberry Pis into a cluster. This includes defining the networking within the cluster and to an external network.
* [Install Kubernetes on a Raspberry Pi Cluster](https://medium.com/better-programming/install-kubernetes-on-a-raspberry-pi-cluster-49ad9a762d08): This includes installing & configuring Kubernetes as well as building and deploying a simple service. This latter step involves a minimal application that can be used to test the Kubernetes cluster as well as simple external network access to the application.
* [Install Kubernetes Ingress on a Raspberry Pi Cluster](https://medium.com/@RichYoungkin/install-kubernetes-ingress-on-a-raspberry-pi-cluster-e8d5086c5009): Ingress is a useful way to support external access to services inside the cluster. In this article I’ll be describe how to install, configure, and test Traefik as an Ingress Controller. This article also includes installing and configuring Helm. Helm will be used to deploy Traefik. Helm will also be used in later articles in the series.
* [Kubernetes Application Monitoring on a Raspberry Pi Cluster](https://medium.com/better-programming/kubernetes-application-monitoring-on-a-raspberry-pi-cluster-fa8f2762b00c) — Install Prometheus, Grafana, and an EFK (Elasticsearch/Fluentd/Kibana) as a monitoring stack. It also covers installing Telegraf on the cluster to monitor machine level metrics for hosts on the cluster. E.g., CPU load and temperature, disk usage
* [How to Install MariaDB on a Raspberry Pi](https://medium.com/better-programming/how-to-install-mysql-on-a-raspberry-pi-ad3f69b4a094) — The title says it all. One thing to note, MariaDB is a drop-in replacement for MySQL