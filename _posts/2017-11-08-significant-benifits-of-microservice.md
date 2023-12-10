---
layout: post
title: Significant Benefits of Microservice Architecture Pattern
---

In Microservice architecture there is a number of benefits. I’ll point some major benefits of microservice rather than monolithic approach, what I got from my experience.


### Splitting Problem Complexity:
Microservice architecture pattern tackles the problem complexity by decomposing what would otherwise be a monstrous monolithic application into a set of services. While the total amount of functionality is unchanged, the application has been broken up into manageable chunks or services. Each service has a well-defined boundary in the form of a remote procedure call driven message API(RPC). The microservice architecture pattern enforces a level of modularity that in practice is extremely difficult to achieve with a monolithic code base. We/Developers are able to focus on a specific problem or part of the problem on a specific boundary of a service. On the other hand, Individual services are much faster to develop and easier to understand and maintain.

### Independent Development: 
Microservice architecture enables each service to be developed independently by a team/developer that is focused on that service only. The Developer/team are free to choose whatever technologies make sense, provided that the service honors the API contract. Of course, most organizations would want to avoid complete anarchy by limiting technology options. However, this freedom means that developers are no longer obligated to use the possibly obsolete technologies that existed at the start of a new project. When writing a new service, they have the free option of using current technology. Moreover, since services are relatively small, it becomes more feasible to rewrite an old service using current tool and technology stuff.

### Independent Deployment: 
Microservice architecture pattern enables each service to be deployed independently. Developers never need to coordinate the deployment of changes that are local to their service. These kinds of changes can be deployed as soon as they have been tested. The UI team can perform test and rapidly iterate on UI changes. And the microservice pattern makes well suited with continuous deployment possible.

### Independent Scalability: 
Microservice architecture enables each service to be scaled independently. We can deploy just the number of instances of each service that satisfy its capacity and availability constraints. Moreover, we can use deploy CPU-intensive image processing service on EC2/Azure compute Optimized instances and deploy an in-memory database service on EC2 memory-optimized instances.
