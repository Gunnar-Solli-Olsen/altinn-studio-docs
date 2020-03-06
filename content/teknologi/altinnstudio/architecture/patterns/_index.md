---
title: Architecture Patterns
description: The architecture defines some architecture patterns that is reused accross the different solutions. 
tags: [architecture]
linktitle: Patterns
weight: 7
alwaysopen: false
aliases:
 - /architecture/
---


## Cloud Native
The solutions are designed and built to be ["Cloud Native"](https://github.com/cncf/toc/blob/master/DEFINITION.md), meaning
it is built be scalable applications hosted in public cloud. 

There is serveral Architecture Patterns that Cloud Native applications are built on.

Containers, microservices and declarative APIs exemplify this approach.

These techniques enable loosely coupled components that are resilient, manageable, and observable. 

Combined with robust automation, they allow the devops team to make changes frequently.

The Cloud Native blueprint below show the important aspect of a Cloud Native solution.

{{%excerpt%}}
<object data="/teknologi/altinnstudio/architecture/patterns/cloudnative.svg" type="image/svg+xml" style="width: 100%;";></object>
{{% /excerpt%}}


The following listing describes the different Architecture Patterns selected for Altinn

## Microservice Pattern

### Overview
The [Microservice Pattern](https://en.wikipedia.org/wiki/Microservices) is one of the more defining patterns in the platform.

In the new Altinn Solutions this mean that Altinn Platform will be grouped in functional related components, that will run as 
separate applications in docker containers. 

Example components are authorization and storage.

Each application created in Altinn Studio will be a isolated microservice application with API's and frontend. 

### Conceptual View 

{{%excerpt%}}
<object data="/teknologi/altinnstudio/architecture/patterns/microservices.svg" type="image/svg+xml" style="width: 100%;";></object>
{{% /excerpt%}}

### Benefits 

- Strong Module Boundaries: Microservices reinforce modular structure, which is particularly important for larger teams. 
- Independent Deployment: Simple services are easier to deploy, and since they are autonomous, are less likely to cause system failures when they go wrong. 
- Technology Diversity: With microservices you can mix multiple languages, development frameworks and data-storage technologies.

### Risk & Issues

- Distribution: Distributed systems are harder to program, since remote calls are slow and are always at risk of failure. 
- Eventual Consistency: Maintaining strong consistency is extremely difficult for a distributed system, which means everyone has to manage eventual consistency. 
- Operational Complexity: You need a mature operations team to manage lots of services, which are being redeployed regularly. 

## Layered Pattern

### Overview
Applications/Components are organized into horizontal layers, each layer performing a specific 
role within the application. Typical the layers will typical be Presentation, API, Business and dataacess where dataccess could be access to database or 
API calls to other components.

### Conceptual view 
{{%excerpt%}}
<object data="/teknologi/altinnstudio/architecture/patterns/layered.svg" type="image/svg+xml" style="width: 100%;";></object>
{{% /excerpt%}}

### Benefits
- Testability. Dependency injection. Possible to mock lower layers. 

### Risk & Issues