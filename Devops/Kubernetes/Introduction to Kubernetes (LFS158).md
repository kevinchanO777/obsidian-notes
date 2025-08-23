
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

#### Control Plane Nodes Overview:

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

**Stacked Topology:**
![[Pasted image 20250823191149.png]]

**External Topology:**
![[Pasted image 20250823191203.png]]

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

- **<mark style="background: #ADCCFFA6;">etcd</mark>**: 
```
A distributed, highly available key-value store that is used to persist a Kubernetes cluster's state, including configuration and state information.

New data is written to the data store only by appending to it, data is never replaced in the data store. Obsolete data is compacted (or shredded) periodically to minimize the size of the data store.


```

- <mark style="background: #ADCCFFA6;">cloud-controller-manager</mark> (==optional==): Manages interactions with underlying cloud providers, handling cloud-specific control loops like node and load balancer management.

In addition, the control plane node runs: container runtime, node agent (kubelet), proxy (kube-proxy), optional add-ons for observability, such as dashboard, cluster-level monitoring, and logging.


#### Worker Node:

A worker node has the following components: container runtime, node agent - kubelet, kubelet - CRI shims, proxy - kube-proxy, add-ons (for DNS, observability components such as dashboards, cluster-level monitoring and logging, and device plugins).

<mark style="background: #ADCCFFA6;">Container Runtime</mark>
>In order to manage a container's lifecycle, Kubernetes requires a container runtime on the node where a Pod and its containers are to be scheduled. A runtime is required on each node of a Kubernetes cluster, both control plane and worker. 
>
>The [recommendation](https://kubernetes.io/docs/setup/) is to run the Kubernetes control plane components as containers, hence the necessity of a runtime on the control plane nodes. Kubernetes supports several container runtimes: *CRI-O, containerd, Docker Engine*


<mark style="background: #ADCCFFA6;">Node Agent -  kublet</mark>
>The kubelet is an agent running on each node, control plane and workers, and it communicates with the control plane. It receives Pod definitions, primarily from the API Server, and interacts with the container runtime on the node to run containers associated with the Pod. It also monitors the health and resources of Pods running containers.
>
>The kubelet connects to container runtimes through a plugin based interface - the [Container Runtime Interface](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md) (CRI). The CRI consists of protocol buffers, gRPC API, libraries, and additional specifications and tools. In order to connect to interchangeable container runtimes, kubelet uses a *CRI shim*, an application which provides a clear abstraction layer between kubelet and the container runtime.

**Container Runtime Interface**
![[Pasted image 20250823224320.png]]

<mark style="background: #ADCCFFA6;">kubelet - CRI shims</mark>
>Shims are Container Runtime Interface (CRI) implementations, interfaces or adapters, specific to each container runtime supported by Kubernetes.

For example **cri-containerd** which allows containers to be directly created and managed with containerd at kubelet's request:
![[Pasted image 20250823224929.png]]

<mark style="background: #ADCCFFA6;">Proxy - kube-proxy</mark>
>  The kube-proxy is the network agent which runs on each node, control plane and workers, responsible for dynamic updates and maintenance of all networking rules on the node. It abstracts the details of Pods networking and forwards connection requests to the containers in the Pods.
>  
>  The kube-proxy is responsible for TCP, UDP, and SCTP stream forwarding or random forwarding across a set of Pod backends of an application, and it implements forwarding rules defined by users through Service API objects.
>  
>  The kube-proxy node agent operates in conjunction with the iptables of the node. Iptables is a firewall utility created for the Linux OS that can be managed by users through a CLI utility of the same name. The iptables utility is available for and pre-installed on many Linux distributions.

Add-ons

Add-ons are cluster features and functionality not yet available in Kubernetes, therefore implemented through 3rd-party plugins and services.

- DNS  
    Cluster DNS is a DNS server required to assign DNS records to Kubernetes objects and resources.
- Dashboard  
    A general purpose web-based user interface for cluster management.
- Monitoring  
    Collects cluster-level container metrics and saves them to a central data store.
- Logging  
    Collects cluster-level container logs and saves them to a central log store for analysis.
- Device Plugins  
    For system hardware resources, such as GPU, FPGA, high-performance NIC, to be advertised by the node to application pods.