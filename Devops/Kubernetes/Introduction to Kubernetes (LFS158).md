
# Chapter 1 - From Monolith to Microservices:


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

# Chapter 2 - Container Orchestration

>Container orchestrators are tools which group systems together to form clusters where containers' deployment and management is automated at scale while meeting the requirements as listed:

- Fault-tolerance
- On-demand scalability
- Optimal resource usage
- Auto-discovery to automatically discover and communicate with each other
- Accessibility from the outside world
- Seamless updates/rollbacks without any downtime.

# Chapter 3 - Kubernetes

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

# Chapter 4 - Kubernetes Architecture

At a very high level, Kubernetes is a cluster of compute systems categorized by their distinct roles:

- One or more ==control plane nodes
- One or more ===worker nodes== (optional, but recommended).
 
![[Pasted image 20250821163746.png]]

# Control Plane Nodes Overview:

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

# Control Plane Nodes Components:

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


# Worker Node:

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

# Networking Challenges

Decoupled microservices based applications rely heavily on networking in order to mimic the tight-coupling once available in the monolithic era. Networking, in general, is not the easiest to understand and implement. Kubernetes is no exception - as a containerized microservices orchestrator it needs to address a few distinct networking challenges:

- Container-to-Container communication inside Pods
- Pod-to-Pod communication on the same node and across cluster nodes
- Service-to-Pod communication within the same namespace and across cluster namespaces
- External-to-Service communication for clients to access applications in a cluster

All these networking challenges must be addressed by a Kubernetes cluster and its plugins.


### Pod-to-Pod communication across nodes

In a Kubernetes cluster Pods, groups of containers, are scheduled on nodes in a nearly unpredictable fashion. Regardless of their host node, Pods are expected to be able to communicate with all other Pods in the cluster, all this without the implementation of Network Address Translation (NAT). This is a fundamental requirement of any networking implementation in Kubernetes.

The Kubernetes network model aims to reduce complexity, and it treats Pods as VMs on a network, where each VM is equipped with a network interface - thus each Pod receiving a unique IP address. This model is called "**IP-per-Pod**" and ensures Pod-to-Pod communication, just as VMs are able to communicate with each other on the same network.

Let's not forget about containers though. They share the Pod's network namespace and must coordinate ports assignment inside the Pod just as applications would on a VM, all while being able to communicate with each other on **localhost** - inside the Pod. However, containers are integrated with the overall Kubernetes networking model through the use of the [Container Network Interface](https://www.cni.dev/) (CNI) supported by [CNI plugins](https://github.com/containernetworking/cni#3rd-party-plugins). CNI is a set of specifications and libraries which allow plugins to configure the networking for containers. While there are a few [core plugins](https://github.com/containernetworking/plugins#plugins), most CNI plugins are 3rd-party Software Defined Networking (SDN) solutions implementing the Kubernetes networking model. In addition to addressing the fundamental requirement of the networking model, some networking solutions offer support for Network Policies. [Flannel](https://github.com/coreos/flannel/), [Weave](https://www.weave.works/oss/net/), [Calico](https://www.tigera.io/project-calico/), and [Cilium](https://cilium.io/) are only a few of the SDN solutions available for Kubernetes clusters.

# Chapter 5 - Installing Kubernetes

### Kubernetes Configuration

Kubernetes can be installed using different cluster configurations. The major installation types are described below:

- **All-in-One Single-Node Installation**  
    In this setup, all the control plane and worker components are installed and running on a single-node. While it is useful for learning, development, and testing, it is not recommended for production purposes.
- **Single-Control Plane and Multi-Worker Installation**  
    In this setup, we have a single-control plane node running a stacked etcd instance. Multiple worker nodes can be managed by the control plane node.
- **Single-Control Plane with Single-Node etcd, and Multi-Worker Installation**  
    In this setup, we have a single-control plane node with an external etcd instance. Multiple worker nodes can be managed by the control plane node.
- **Multi-Control Plane and Multi-Worker Installation**  
    In this setup, we have multiple control plane nodes configured for High-Availability (HA), with each control plane node running a stacked etcd instance. The etcd instances are also configured in an HA etcd cluster and multiple worker nodes can be managed by the HA control plane.
- **Multi-Control Plane with Multi-Node etcd, and Multi-Worker Installation**  
    In this setup, we have multiple control plane nodes configured in HA mode, with each control plane node paired with an external etcd instance. The external etcd instances are also configured in an HA etcd cluster, and multiple worker nodes can be managed by the HA control plane. This is the most advanced cluster configuration recommended for production environments.

#### Installing Local Learning Clusters

There are a variety of installation tools allowing us to deploy single- or multi-node Kubernetes clusters on our workstations, for learning and development purposes. For example: **Minikube, Kind, Docker Desktop, etc...**

### Installing Production Clusters with Deployment Tools

When it comes to production ready solutions, there are several recommended tools for Kubernetes cluster bootstrapping and a few that are also capable of provisioning the necessary hosts on the underlying infrastructure.

1. **kubeadm**
2. **kubespray**
3. **kops**

In addition, for a manual installation approach, the [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) GitHub project by [Kelsey Hightower](https://twitter.com/kelseyhightower) is an extremely helpful installation guide and resource. The project aims to teach all the detailed steps involved in the bootstrapping of a Kubernetes cluster, steps that are otherwise automated by various tools mentioned in this chapter and obscured from the end user.


### Production Clusters from Certified Solutions Providers

**Hosted Solutions**  
Hosted Solutions providers fully manage the provided software stack, while the user pays hosting and management charges. Popular vendors providing hosted solutions for Kubernetes are (listed in alphabetical order):

- [Alibaba Cloud Container Service for Kubernetes](https://www.alibabacloud.com/product/kubernetes) (ACK)
- [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/) (EKS)
- [Azure Kubernetes Service](https://azure.microsoft.com/en-us/services/kubernetes-service/) (AKS)

# Chapter 6 - Installing Minikube

Minikube is one of the easiest, most flexible and popular methods to run an all-in-one or a multi-node local Kubernetes cluster directly on our local workstations.

```sh

# Start a k8s cluster (optional --driver=docker)
minikube start

# Custom minikube profile
minikube start --nodes=2 --kubernetes-version=v1.28.1 --driver=docker --profile doubledocker

# Check the status of the Minikube cluster
minikube status

# Temporary stop all applications running in Minikube
minikube stop

# Remove Minikube cluster
minikube delete

# View the status of all clusters in a table formatted output
minikube profile list

# Switch profile
minikube profile <profile_name>

minikube node list
minikube ip

```

The **minikube start** by default selects a driver isolation software, such as a hypervisor or a container runtime, if one (VitualBox) or multiple are installed on the host workstation. In addition it downloads the latest Kubernetes version components. With the selected driver software it provisions a single VM named _minikube_ (with hardware profile of CPUs=2, Memory=6GB, Disk=20GB) or container (Docker) to host the default single-node all-in-one Kubernetes cluster.

The **minikube start** command generates a default minikube cluster with the specifications described above and it will store these specs so that we can restart the default cluster whenever desired. The object that stores the specifications of our cluster is called a _profile_.

# Chapter 7 - Accessing Minikube

Any healthy running Kubernetes cluster can be accessed via any one of the following methods: Command Line Interface (CLI) tools and scripts, web-based User Interface (Web UI) from a web browser, APIs from CLI or programmatically.

### kubectl

**kubectl** allows us to manage local Kubernetes clusters like the Minikube cluster, or remote clusters deployed in the cloud. It is generally installed before installing and starting Minikube, but it can also be installed after the cluster bootstrapping step.

```
kubectl version --client
kubectl version --client --output=yaml

kubectl get po -A
kubectl get nodes
kubectl get namespaces

kubectl cluster-info

kubectl config view
```

While starting Minikube, the startup process creates, by default, a configuration file, **config**, inside the **.kube** directory (often referred to as the **[kubeconfig](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)**), which resides in the user's **home** directory (~/.kube/config).

Although for the Kubernetes cluster installed by Minikube the **~/.kube/config** file gets created automatically, this is not the case for Kubernetes clusters installed by other tools. In other cases, the config file has to be created manually and sometimes re-configured to suit various networking and client/server setups.


### Kubernetes Dashboard

To access the dashboard from Minikube, we can use the **minikube dashboard** command, which opens a new tab in our web browser displaying the Kubernetes Dashboard, but only after we list, enable required addons, and verify their state:

```sh
minikube addons list

minikube addons enable metrics-server

minikube addons enable dashboard

minikube addons list

minikube dashboard
```


### APIs with 'kubectl proxy'

Issuing the **kubectl proxy** command, **kubectl** <mark style="background: #BBFABBA6;">authenticates</mark> with the API server on the control plane node and makes services available on the default proxy port **8001**.

First, we issue the **kubectl proxy** command:

```sh
# Starting to serve on 127.0.0.1:8001
kubectl proxy
```

To get all api endpoints:
```sh
curl http://localhost:8001/
```

### APIs with Authentication

When not using the **kubectl proxy**, we need to authenticate to the API Server when sending API requests. We can authenticate by providing a Bearer Token when issuing a **curl** command, or by providing a set of keys and certificates.

A **Bearer Token** is an access token that can be generated by the authentication server (the API Server on the control plane node) at the client's request. 

Using that token, the client can securely communicate with the Kubernetes API Server without providing additional authentication details, and then, access resources. The token may need to be provided again for subsequent resource access requests.