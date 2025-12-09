---
title: Technical Architecture for Large Scale Websites Study Notes
date: 2021-01-03 11:26:02
tags: [Site Architecture]
categories:
 - backend
---

# 1 Overview of Architecture Patterns

## 1.1 Layering

In large-scale website architecture, a layered architecture is used to slice the website software system in the horizontal aspect into application layer, service layer, and data layer. This is crucial for the website to support high concurrency and develop in the distributed direction afterwards.

* Application layer: specific operations and view presentation. (Front-end)
* Service layer: service support for the application layer. (back-end)
* Data layer: provide data storage access services.

<!-- more -->

## 1.2 Segmentation

Segmentation, that is, slicing and dicing the website software system in the vertical aspect. For example, at the application layer, different services are segmented; at the service layer, services are segmented into appropriate modules.

## 1.3 Distributed

One of the main purposes of layering and segmentation is to facilitate distributed deployment of the sliced modules, i.e., different modules are deployed on different servers and work together through remote calls. The common distributed schemes are:

* Distributed applications and services. Improves site performance, speeds development, and enables reuse of common services across applications.
* Distributed static resources. Static resources are independently distributed deployment, using a separate domain name, that is, separation of dynamic and static.
* Distributed data and storage. Distributed deployment of relational databases, NoSQL distributed databases.
* Distributed computing. Hadoop and MapReduce distributed computing framework batch computing.

This also places a higher demand on the development and maintenance of the website.

## 1.4 Clustering

After the use of distributed, for the user to access the centralized module, but also need to independently deployed server clustering, that is, multiple servers deploying the same application constitute a cluster, through the load balancing equipment to provide services to the outside world, but also to improve the availability of the system.

## 1.5 Caching

Used to improve software performance. Included:

* CDN: Content Delivery Network, which caches some static resources (e.g., hot videos with high access in video websites) and returns them to users from the nearest network service provider.
* Reverse proxy: caches static resources of a website to achieve load balancing.
* Local Cache: caches hot data locally in the application server, which is directly accessed by the application program without accessing the database.
* Distributed Cache: Cached data requires an increase in memory space, which requires distributed caching, where the application accesses the cached data through network communication.

## 1.6 Asynchronous

In large-scale website architecture, the means of system decoupling, in addition to layering, partitioning, distribution, etc., there is also asynchrony. Asynchrony can be realized within a single server by means of multi-threaded shared memory queues; in distributed systems, multiple server clusters realize asynchrony through distributed message queues. Asynchronous architectures are typically producer-consumer patterns.
Using asynchronous message queues improves system availability, speeds up website response, and eliminates concurrent access spikes
Asynchronous message queuing is a typical producer-consumer pattern.

## 1.7 Redundancy

In order for the website to continue to be served in the event of a server failure, a certain degree of server redundancy is required (at least two), and data redundancy backups (regular backups cold backups, master-slave separation hot backups).

## 1.8 Automation

Reduce failures by reducing human intervention. Includes: release process automation, automated code management, automated testing, automated security testing, automated deployment, automated monitoring, automated alerting, automated failover, automated failback, automated degradation, automated resource allocation.

## 1.9 Security

Authentication by password and cell phone check code; network communication for sensitive operations needs to be encrypted; prevent ddos, use captcha identification; prevent XSS attacks, SQL injection, escape and other processing; filter spam and so on.