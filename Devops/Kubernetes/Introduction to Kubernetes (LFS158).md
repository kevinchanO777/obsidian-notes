
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

#### Chapter 4 - Kubernetes Architecture

At a very high level, Kubernetes is a cluster of compute systems categorized by their distinct roles:

- One or more ==control plane nodes
- One or more ===worker nodes== (optional, but recommended).
 
![[Pasted image 20250821163746.png]]

##### Control Plane Nodes Overview:

- **Control Plane Node Function**:
    - Provides a running environment for control plane agents.
    - Manages the state of a Kubernetes cluster, acting as the brain for all cluster operations.

- **Control Plane Components**:
    - Consist of agents with distinct roles in cluster management.
    - Users interact with the cluster via CLI, Web UI Dashboard, or API.

- **Importance of Control Plane**:
    - Critical to keep running to avoid downtime and service disruption.

- **High-Availability (HA) Configuration**:
    - Adding control plane node replicas enhances fault tolerance.
    - ==Only one node actively manages the cluster==, while others stay in sync for resiliency.

- **Cluster State Storage**:
    - Cluster configuration data stored in a distributed key-value store.
    - ==Stores only cluster state data==, not client workload data.


- **Key-Value Store Topologies**:
    - ==**Stacked Topology**==: Key-value store on the ==control plane node==; HA replicas ensure resiliency.
    - **==External Topology==**: Key-value store on a ==dedicated host==, reducing data loss risk but requiring separate replication for HA.
    - **External topology**<mark style="background: #BBFABBA6;"> increases resiliency</mark> but requires <mark style="background: #FF5582A6;">additional hardware</mark>, raising operational costs.


##### Control Plane Nodes Components:

A control plane node runs the following essential control plane components and agents: 

>**API server, scheduler, controller managers, and key-value data store.**

- <mark style="background: #ADCCFFA6;">kube-apiserver</mark>: 
```
The core server that exposes the Kubernetes API via HTTP, handling all REST requests and serving as the front-end for the control plane. 

The API Server is the only control plane component to talk to the key-value store
```

- <mark style="background: #ADCCFFA6;">kube-scheduler</mark>: 
```
Watches for newly created Pods without an assigned node and selects the most suitable node for them based on resource requirements, policies, and constraints.
```

- <mark style="background: #ADCCFFA6;">kube-controller-manager</mark>: 
```
Runs multiple controller processes (e.g., node controller, replication controller) that regulate the state of the cluster to match the desired state defined in objects' configuration data via API server.
```


- **<mark style="background: #ADCCFFA6;">etcd</mark>**: A distributed, highly available key-value store that persists all cluster data, including configuration and state information.


- **cloud-controller-manager** (==optional==): Manages interactions with underlying cloud providers, handling cloud-specific control loops like node and load balancer management.

In addition, the control plane node runs: container runtime, node agent (kubelet), proxy (kube-proxy), optional add-ons for observability, such as dashboard, cluster-level monitoring, and logging.