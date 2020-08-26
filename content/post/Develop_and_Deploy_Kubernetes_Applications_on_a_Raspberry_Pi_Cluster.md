---
title: "Develop and Deploy Kubernetes Applications on a Raspberry Pi Cluster"
date: 2020-08-26T01:01:31-06:00
draft: false
image: "images/RaspberryPiClusterDark.jpeg"
tags: ["Raspberry Pi", "Docker", "Kubernetes", "Helm", "fluentd", "Networking", "Kubernetes Ingress", "MySQL", "Grafana", "Prometheus", "Elasticsearch", "Traefik"]
categories: ["Raspberry Pi", "Kubernetes"]
---

Want to set up a Kubernetes lab, but don't want use Minikube or spend a lot of money on hardware. This is the first post in a series that describes how to set up a complete Kubernetes cluster on a cluster of Raspberry Pis. This will include topics such as setting up a logging infrastructure, monitoring, Ingress, and other platform level capabilities.
<!--more-->

---

## The full text of this article can be found on [Medium](https://medium.com/better-programming/develop-and-deploy-kubernetes-applications-on-a-raspberry-pi-cluster-fbd4d97a904c?source=friends_link&sk=df18f8cdfc8b90aa25b2b6676346d1ec)

---

# Overview

This article is the first in a series that follows my journey to develop and deploy Kubernetes and a sample microservices style application on a Raspberry Pi cluster.

Parts of this series are intended to be a meta-resource for developing and deploying Kubernetes applications on a Raspberry Pi cluster. Common tasks such as deploying an OS on Raspberry Pi are readily available in other resources. I won’t replicate information in this series. This series will provide details in areas that aren’t not present in other sources.

Tasks in this series that weren’t performed on a Raspberry Pi (e.g. the initial setup and various admin tasks) were performed on a MacBook Pro running Catalina.

## Articles in the Series

The full set of articles are:

* [Set up a Raspberry Pi Cluster](https://medium.com/better-programming/setup-a-raspberry-pi-cluster-ff484a1c6be9) : Describes how to set up and configure a group of Raspberry Pis into a cluster. This includes defining the networking within the cluster and to an external network.
* [Install Kubernetes on a Raspberry Pi Cluster](https://medium.com/better-programming/install-kubernetes-on-a-raspberry-pi-cluster-49ad9a762d08): This includes installing & configuring Kubernetes as well as building and deploying a simple service. This latter step involves a minimal application that can be used to test the Kubernetes cluster as well as simple external network access to the application.
* [Install Kubernetes Ingress on a Raspberry Pi Cluster](https://medium.com/@RichYoungkin/install-kubernetes-ingress-on-a-raspberry-pi-cluster-e8d5086c5009): Ingress is a useful way to support external access to services inside the cluster. In this article I’ll be describe how to install, configure, and test Traefik as an Ingress Controller. This article also includes installing and configuring Helm. Helm will be used to deploy Traefik. Helm will also be used in later articles in the series.
* [Kubernetes Application Monitoring on a Raspberry Pi Cluster](https://medium.com/better-programming/kubernetes-application-monitoring-on-a-raspberry-pi-cluster-fa8f2762b00c) — Install Prometheus, Grafana, and an EFK (Elasticsearch/Fluentd/Kibana) as a monitoring stack. It also covers installing Telegraf on the cluster to monitor machine level metrics for hosts on the cluster. E.g., CPU load and temperature, disk usage
* [How to Install MariaDB on a Raspberry Pi](https://medium.com/better-programming/how-to-install-mysql-on-a-raspberry-pi-ad3f69b4a094) — The title says it all. One thing to note, MariaDB is a drop-in replacement for MySQL