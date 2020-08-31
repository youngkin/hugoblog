---
title: "Kubernetes Application Monitoring on a Raspberry Pi Cluster"
description: "This article is about monitoring applications deployed to Kubernetes running on Raspberry Pis."
date: 2020-08-26T13:06:44-06:00
draft: false
image: "images/Cockpit.jpeg"
tags: ["Raspberry Pi", "Grafana", "Prometheus", "Elasticsearch", "Kubernetes", "Monitoring", "Metrics", "Logging"]
categories: ["Raspberry Pi", "Kubernetes"]
---



> Photo by [Andres Dallimonti](https://unsplash.com/@dallimonti?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/sailor?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

## The full text of this article can be found on [Medium](https://medium.com/better-programming/kubernetes-application-monitoring-on-a-raspberry-pi-cluster-fa8f2762b00c?source=friends_link&sk=81553ee8e61a841e43143a8e73c1bb9e)

---

This article is about monitoring applications deployed to a Kubernetes cluster running on Raspberry Pis, specifically the monitoring infrastructure.

It’s important to note that while the focus of this article is the Raspberry Pi platform, much of what’s described, except for the ARM specific areas, is applicable to other hardware platforms.

This is the fifth article in the [Develop and Deploy Kubernetes Applications on a Raspberry Pi Cluster](https://medium.com/better-programming/develop-and-deploy-kubernetes-applications-on-a-raspberry-pi-cluster-fbd4d97a904c?source=friends_link&sk=df18f8cdfc8b90aa25b2b6676346d1ec) series. The previous article, [Installing Kubernetes Ingress on a Raspberry Pi Cluster](https://medium.com/better-programming/install-kubernetes-ingress-on-a-raspberry-pi-cluster-e8d5086c5009?source=friends_link&sk=148c7e4d0c276bad20fa7f2ce902736e) is a prerequisite and must be completed before performing the steps described in this article.

## Overview

Any production-grade application needs a good monitoring stack. For most applications, this will include a mix of logging and metrics.

Metrics are recorded by an application to track things like request rate, error rate, and request duration, also called [RED](https://www.weave.works/blog/the-red-method-key-metrics-for-microservices-architecture/). Metrics are good for quickly identifying problems — e.g., an excessive error rate or an unacceptable request duration. Metrics don’t identify what caused a particular problem, only that a problem event occurred. Prometheus is one framework that provides an application the ability to record and store metrics at a relatively low cost.

Logging typically involves writing records to a file or stdout. Log records usually include things like timestamps and details about an event that occurred. For example, a log record might contain information about a malformed HTTP request and include information about the requesting client as well as information about what specifically was wrong with the request. As such, logs are very good for investigating and troubleshooting problems.

Both metrics and logging have a place in application monitoring. Metrics are relatively cheap and are easy to alert on. Metrics have a downside relative to logging in that they contain no context, just information that something happened. In contrast, logs can provide the context needed to investigate and troubleshoot a problem. The downside to logging is all that context is expensive relative to metrics. Metrics are very small, often just a label, a number, and a timestamp. Logs can be quite voluminous, and it can be relatively hard to alert from a log record.

There are a number of resources that cover the philosophy and mechanics of monitoring. Two excellent resources are Google’s SRE guide, specifically [Chapter 6: Monitoring Distributed Systems](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems/). Weaveworks has an excellent [write-up on using metrics with Prometheus and Grafana](https://www.weave.works/docs/cloud/latest/tasks/monitor/best-instrumenting/). I’ve included other resources as well at the end of this article.

## Articles in the Series

The full set of articles are:

* [Set up a Raspberry Pi Cluster](https://medium.com/better-programming/setup-a-raspberry-pi-cluster-ff484a1c6be9) : Describes how to set up and configure a group of Raspberry Pis into a cluster. This includes defining the networking within the cluster and to an external network.
* [Install Kubernetes on a Raspberry Pi Cluster](https://medium.com/better-programming/install-kubernetes-on-a-raspberry-pi-cluster-49ad9a762d08): This includes installing & configuring Kubernetes as well as building and deploying a simple service. This latter step involves a minimal application that can be used to test the Kubernetes cluster as well as simple external network access to the application.
* [Install Kubernetes Ingress on a Raspberry Pi Cluster](https://medium.com/@RichYoungkin/install-kubernetes-ingress-on-a-raspberry-pi-cluster-e8d5086c5009): Ingress is a useful way to support external access to services inside the cluster. In this article I’ll be describe how to install, configure, and test Traefik as an Ingress Controller. This article also includes installing and configuring Helm. Helm will be used to deploy Traefik. Helm will also be used in later articles in the series.
* [Kubernetes Application Monitoring on a Raspberry Pi Cluster](https://medium.com/better-programming/kubernetes-application-monitoring-on-a-raspberry-pi-cluster-fa8f2762b00c) — Install Prometheus, Grafana, and an EFK (Elasticsearch/Fluentd/Kibana) as a monitoring stack. It also covers installing Telegraf on the cluster to monitor machine level metrics for hosts on the cluster. E.g., CPU load and temperature, disk usage
* [How to Install MariaDB on a Raspberry Pi](https://medium.com/better-programming/how-to-install-mysql-on-a-raspberry-pi-ad3f69b4a094) — The title says it all. One thing to note, MariaDB is a drop-in replacement for MySQL