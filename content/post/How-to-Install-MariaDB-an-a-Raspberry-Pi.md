---
title: "How to Install MariaDB on a Raspberry Pi"
description: "This article covers installing MariaDB, a MySQL compatible DBMS, on a Raspberry Pi"
date: 2020-08-26T13:07:25-06:00
draft: false
image: "images/IndexCardsDark.jpeg"
tags: ["Raspberry Pi", "MySQL", "MariaDB"]
categories: ["Raspberry Pi"]
GHissueID: 1
toc: false
---

This article covers how to install and configure [MariaDB](https://mariadb.org/), a feature-equivalent alternative to MySQL, on a Raspberry Pi. Installing is relatively straightforward. Configuring the database is somewhat more involved but still quite manageable.

## The full text of this article can be found on [Medium](https://medium.com/better-programming/how-to-install-mysql-on-a-raspberry-pi-ad3f69b4a094?source=friends_link&sk=d4fdd7c2a467b2ac2e6beaded27365bd)

---

This is the sixth and final article in the [Develop and Deploy Kubernetes Applications on a Raspberry Pi Cluster](https://medium.com/better-programming/develop-and-deploy-kubernetes-applications-on-a-raspberry-pi-cluster-fbd4d97a904c) series. It marks the transition from a purely Raspberry Pi focus to software development more generally. With the installation of a database, we will have a full-featured platform that meets our requirements for application development and deployment.

## Articles in the Series

The full set of articles are:

* [Set up a Raspberry Pi Cluster](https://medium.com/better-programming/setup-a-raspberry-pi-cluster-ff484a1c6be9) : Describes how to set up and configure a group of Raspberry Pis into a cluster. This includes defining the networking within the cluster and to an external network.
* [Install Kubernetes on a Raspberry Pi Cluster](https://medium.com/better-programming/install-kubernetes-on-a-raspberry-pi-cluster-49ad9a762d08): This includes installing & configuring Kubernetes as well as building and deploying a simple service. This latter step involves a minimal application that can be used to test the Kubernetes cluster as well as simple external network access to the application.
* [Install Kubernetes Ingress on a Raspberry Pi Cluster](https://medium.com/@RichYoungkin/install-kubernetes-ingress-on-a-raspberry-pi-cluster-e8d5086c5009): Ingress is a useful way to support external access to services inside the cluster. In this article I’ll be describe how to install, configure, and test Traefik as an Ingress Controller. This article also includes installing and configuring Helm. Helm will be used to deploy Traefik. Helm will also be used in later articles in the series.
* [Kubernetes Application Monitoring on a Raspberry Pi Cluster](https://medium.com/better-programming/kubernetes-application-monitoring-on-a-raspberry-pi-cluster-fa8f2762b00c) — Install Prometheus, Grafana, and an EFK (Elasticsearch/Fluentd/Kibana) as a monitoring stack. It also covers installing Telegraf on the cluster to monitor machine level metrics for hosts on the cluster. E.g., CPU load and temperature, disk usage
* [How to Install MariaDB on a Raspberry Pi](https://medium.com/better-programming/how-to-install-mysql-on-a-raspberry-pi-ad3f69b4a094) — The title says it all. One thing to note, MariaDB is a drop-in replacement for MySQL