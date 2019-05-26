---
layout: post
published: true
title: Microservices with Azure Service Fabric
subtitle: ...An Overview
date: '2019-05-27'
tags:
  - Azure
  - Service fabric
  - Microservices
  - Containers
---
In this blog, I’ll try to explain the various options to develop and run microservices on Azure Service fabric along with some details about Azure Service Fabric itself. 

> Note: This blog should NOT be viewed as a comprehensive article on Azure Service Fabric.

Traditionally, applications are composed of multiple tiers which produce a single logical executable. A typical application will have a Client-side user interface layer, Server-side business layer which handles the business logics and a database layer which controls the database operations. These are Monolithic applications in which the components are tightly coupled like Lego blocks. If one component needs a change, often several other components in the application are impacted and requires a change.
Despite this, monolithic applications pose several other problems like

- Entire application needs a deployment even for a small update

- Often needs entire application testing for a small update as impact of the change is not known

- A small bug can bring down the entire application

- Scaling these applications is a challenge due to various resource constraints. Because you must scale entire application resources and not just the ones that need

- Drastically increases the start-up time in complex applications

- It gets tedious to adopt new technologies as it will affect the entire application and will be difficult to maintain

With ever growing list of problems with monolithic architecture, comes microservices architecture to solve these problems. The shift from Monolithic architecture to microservices architecture is one of the major breakthroughs in the IT industry in recent years. Since last couple of years, architects stopped questioning whether, the microservices is a good pattern for software development and started thinking of how to best implement the microservices pattern.

In Microservices architecture the application is divided into small services which usually handles a certain business domain. These services are loosely coupled and can be scaled, deployed and maintained independently.

With increasing adoption of the microservices architecture, a need for a platform to maintain these services has naturally evolved. 

# **Azure Service Fabric**

Service fabric is a distributed platform that makes it easy to package, deploy and manage scalable and reliable microservices and containers. Some of the Core Azure offerings and other Microsoft services like Azure SQL Database, Azure IoT Hub, Microsoft Power BI, Azure Event Hub, Dynamics 365 are all built on Service Fabric platform. 

It is composed of a shared pool of machines(cluster) which can run high density of microservices. You can run 1000’s of services and manage it at a single place.

You can start small and grow based on need. All you need to do is tell Service Fabric how many instances of a service you want, how much compute power and storage these services need, and Service Fabric takes care of spinning up virtual machines, launching services, and so on.

Service fabric makes sure your application is always up and running. Let’s say you update a component of your application and have Service Fabric roll that into production. If the health check fails, Azure Service Fabric automatically rolls back the application to the previous version.

Service fabric is also a container orchestrator using which containers can be provisioned, scheduled and managed at very high scale. 

Service fabric internally stores every service in multiple instances in separate partitions/replicas. if one instance fails then the service is served from a different instance and service fabric rebuilds the failed instance. 

# **Building services for service fabric**

The applications that runs in service fabric cluster should follow a certain programming model which enables the platform to effectively manage the life cycle of these services. Following these programming models to develop services will also greatly simplify the process of developing microservices. Having said that, there is also a way to run a compiled executable of any language or framework with some restrictions. Let’s see the various options to build services that can run on service fabric.

## **Reliable Services**

Reliable services is a simple framework for writing services that integrates with service fabric. A reliable service can be either stateless or stateful service. Reliable service can target either .NET or .NET Core runtime.

A service in which the state(data) is stored in an external data sources like Table Storage, Azure Cosmos DB is a stateless service.

A service in which the state is stored in the service itself is a stateful service. But service fabric is not versatile when it comes to the state storage. The state can only be persisted in one of the following Reliable collections.

- **Reliable Dictionary**
Asynchronous collection of key/value pair similar to Concurrent Dictionary in which key and value can be of any type.
- **Reliable Queue**
Asynchronous strict First-In-First-Out (FIFO) collection similar to Concurrent Queue, the value can be of any type.
- **Reliable Concurrent Queue**
Asynchronous First-In-First-Out (FIFO) collection for high throughput like Concurrent Queue, the value can be of any type. But the FIFO cannot be guaranteed in this collection.

## **Reliable Actor**

Reliable Actor is a Service Fabric implementation of Actor Pattern. In Actor pattern, large number of isolated, independent unit of logic and state called Actor are executed parallelly. 

Reliable Actor is nothing but a Stateful reliable service implementing an Actor Pattern.

This is great for handling many small parallel and independent operations. For e.g. Consider an e commerce site, for every customer who visit the site, you could create one shopping basket actor. Here the actor co locates the data (items added to cart) along with the logic. 

If your logic is not independent and should interact with external components like database, other services or even if you want to access state of other actors then Reliable Actor is probably not the right choice.

One of the important things to notice with Reliable Actor is that it can be used to achieve concurrency by creating many actors running in parallel, but with in a single actor it enforces single threaded execution.

## **Guest Executables**

Perhaps, according to me this is one of the most important service fabric programming models to consider if you must move legacy applications to cloud.

Imagine you have certain legacy components written in any language which you want to reuse while moving the application to cloud. Then simply publish the executable of the legacy component as Guest Executables in Service Fabric. Service Fabric runs this as a Service provides outright features of the platform like High availability, monitoring, application life cycle management and many more.

## **ASP .NET Core**

Service fabric allows you to run any Stateless or Stateful ASP.NET Core application using Reliable Collections as a service. Basically, when you create a ASP.NET Core application for Service Fabric, it is nothing but a Reliable Service. 

Service Fabric gives you the flexibility to choose to run ASP.NET Core application or any service fabric service on either .NET Core or .NET Framework. And suppose if you have an ASP.NET Core application that is already written, you can run it as Guest Executable without rewriting it using Reliable Service Framework.

## **Containers**

Service fabric allows you to deploy containers as a service. However, you will get the flexibility to mix both containers and other services in the same application. 

Suppose if you have an ASP.NET WebAPI application and want to run it as service in Service Fabric, you can containerize it and run it. 

Containers are especially useful to run long running applications or the application that bring down the performance of other services. Because Service Fabric provides Resource Governance feature to containers where you can limit the resources available to the containers.

# **Why Service Fabric?**

- Service fabric can run on any OS or any cloud. You can set up Service Fabric in Azure, AWS or other public clouds. Service Fabric can also run on-premises on Windows or Linux servers. So, write services for Service Fabric once and run anywhere.
- Service Fabric is a data aware platform for low latency and high throughput services.
- Service Fabric can run both windows and Linux containers.
- Service fabric can run executables written in any language as a service
- You don’t have to use Service Fabric by opting to use containers or guest executables. This brings down the learning curve for the developers to some extent.
- Service fabric is an efficient way to lift, shift legacy application into the cloud. It greatly simplifies, microservices development.

# **Pain points in Azure Service Fabric**

- Azure Service Fabric dashboard doesn’t offer much features.
- Service fabric gives nonuser friendly error messages which makes it difficult for user to understand the issue.
- Limited data structures (Reliable collections) for state persistence in Stateful services.
- Platform lock in. Once you choose to develop services to run on Service Fabric, it involves a huge effort to come out of it.

# **Conclusion**

With the traction of Azure Kubernetes Service for container orchestration, I see Azure Service Fabric as a good microservices platform than a container orchestrator. 

In my opinion, although some cons mentioned above bring some pain to the developers, some of the outright features discussed above makes it a go to platform for microservices development specially to lift, shift legacy applications and run it in a microservices architecture.













