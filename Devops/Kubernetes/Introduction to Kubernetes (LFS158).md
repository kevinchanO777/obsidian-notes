
#### Chapter 1 - From Monolith to Microservices:


- Explain what a monolith is.
```
-> Monolithic architecture is a design approach where an application is built as a single, unified unit, with all its components—such as the user interface, business logic, and data access layer—tightly integrated into a single codebase and deployed as one executable unit
```

- Discuss the monolith's challenges in the cloud.

```

Scaling
Availability i.e. up-time
How to test individual component?

```


- Explain the concept of microservices.
- Discuss microservices advantages in the cloud.
- Describe the transformation path from a monolith to microservices.
```
# Who are the worst candidates to refactor into microservieces?

-> Applications tightly coupled with data stores are also poor candidates for refactoring.

-> Better re build again from the ground up.
```

#### Chapter 2 - Container Orchestration

>Container orchestrators are tools which group systems together to form clusters where containers' deployment and management is automated at scale while meeting the requirements as listed:

- Fault-tolerance
- On-demand scalability
- Optimal resource usage
- Auto-discovery to automatically discover and communicate with each other
- Accessibility from the outside world
- Seamless updates/rollbacks without any downtime.

#### Chapter 3 - Kubernetes

> Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications"

##### Supported Features:

- **Automatic bin packing**  
    Kubernetes automatically schedules containers based on resource needs and constraints, to maximize utilization without sacrificing availability.
    
- **Designed for extensibility**  
    A Kubernetes cluster can be extended with new custom features without modifying the upstream source code.
    
- **Self-healing**  
    Kubernetes automatically replaces and reschedules containers from failed nodes. It terminates and then restarts containers that become unresponsive to health checks, based on existing rules/policy. It also prevents traffic from being routed to unresponsive containers.
    
- **Horizontal scaling**  
    Kubernetes scales applications manually or automatically based on CPU or custom metrics utilization.
    
- **Service discovery and load balancing**  
    Containers receive IP addresses from Kubernetes, while it assigns a single Domain Name System (DNS) name to a set of containers to aid in load-balancing requests across the containers of the set.
    
- **Automated rollouts and rollbacks**  
    Kubernetes seamlessly rolls out and rolls back application updates and configuration changes, constantly monitoring the application's health to prevent any downtime.
    
- **Secret and configuration management**  
    Kubernetes manages sensitive data and configuration details for an application separately from the container image, in order to avoid a rebuild of the respective image. Secrets consist of sensitive/confidential information passed to the application without revealing the sensitive content to the stack configuration, like on GitHub.
    
- **Storage orchestration**  
    Kubernetes automatically mounts software-defined storage (SDS) solutions to containers from local storage, external cloud providers, distributed storage, or network storage systems.
    
- **Batch execution**  
    Kubernetes supports batch execution, long-running jobs, and replaces failed containers.
    
- **IPv4/IPv6 dual-stack**  
    Kubernetes supports both IPv4 and IPv6 addresses.