
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

> [!WARNING] It's highly recommended you start with AT LEAST 1 worker node and add more on the go

Minikube is one of the easiest, most flexible and popular methods to run an all-in-one or a multi-node local Kubernetes cluster directly on our local workstations.

```sh

# Start a k8s cluster (optional --driver=docker)
minikube start

# Custom minikube profile
minikube start --nodes=2 --kubernetes-version=v1.33.1 --driver=docker --profile lfs158

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

# View resouces usage
kubectl top po
kubectl top node

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


Let's create an access token for the **<mark style="background: #ADCCFFA6;">default</mark>** ServiceAccount, and grant special permission to access the root directory of the API (special permission that was not necessary when the **kubectl proxy** was used earlier). The special permission will be set through a Role Based Access Control (RBAC) policy. The policy is the <mark style="background: #ADCCFFA6;">clusterrole</mark> defined below, which is granted through the **<mark style="background: #ADCCFFA6;">clusterrolebinding</mark>** definition (RBAC, clusterroles, and clusterrolebindings will be discussed in a later chapter). The special permission is only needed to access the root directory of the API, but not needed to access **/api**, **/apis**, or other subdirectories:

```sh
# Create token named `default`
export TOKEN=$(kubectl create token default)

# Create clusterrole api-access-root
kubectl create clusterrole api-access-root --verb=get --non-resource-url='*'

# View created clusterrole
kubectl describe clusterrole api-access-root

# Grants a ClusterRole across the entire cluster (all namespaces)
kubectl create clusterrolebinding api-access-root --clusterrole=api-access-root --serviceaccount=default:default

# Get address of the api server
export APISERVER=$(kubectl config view | grep https** **|** **cut -f 2- -d ":" | tr -d " ")

# Test
curl $APISERVER --header "Authorization:** **Bearer $TOKEN" --insecure
```

Newly created cluster role shown in the web ui
![[Screenshot 2025-08-25 at 1.12.34 AM.png]]


To delete a cluster role: `kubectl delete clusterrole <clusterrole>`

# Chapter 8 - Kubernetes Building Blocks

Examples of Kubernetes object types are **Nodes, Namespaces, Pods, ReplicaSets Deployments, DaemonSets, etc.**

With each object, we declare our <mark style="background: #ADCCFFA6;">intent</mark>, or the desired state of the object, in the <mark style="background: #ADCCFFA6;">spec</mark> section. The Kubernetes system manages the **status** section for objects, where it records the actual state of the object. At any given point in time, the Kubernetes Control Plane tries to match the object's actual state to the object's desired state.

When creating an object, the object's configuration data section from below the **spec** field has to be submitted to the Kubernetes API Server. The API request to create an object must have the **spec** section, describing the desired state, as well as other details.

### Nodes

Nodes are virtual identities assigned by Kubernetes to systems (Virtual Machines, bare-metal, Containers) in the cluster. They are unique identities used for resource accounting, monitoring, and workload management.

**Node Components**:

- Managed by two Kubernetes node agents: **kubelet** and **kube-proxy**.
- Host a **container runtime** to run containerized workloads (control plane agents and user workloads).
- **Kubelet**: Interacts with the runtime to run containers, monitors container and node health, reports issues and node state to the API Server.
- **Kube-proxy**: Manages network traffic to containers.

**Types of Nodes**:

- **Control Plane Nodes**:
    - Run control plane agents (API Server, Scheduler, Controller Managers, etcd), kubelet, kube-proxy, container runtime, and add-ons (networking, monitoring, logging, DNS).
    - At least one control plane node required; multiple nodes used for High Availability (HA).
- **Worker Nodes**:
    - Run kubelet, kube-proxy, container runtime, and add-ons (networking, monitoring, logging, DNS).
    - Provide resource redundancy in the cluster.
- **Hybrid/Mixed Nodes**:
	- Single all-in-one node hosting both control plane agents and user workloads.
	- Used in cases where high availability and resource redundancy are not critical.

### Namespaces

**Purpose of Namespaces**:
- Partition a Kubernetes cluster into virtual sub-clusters for multiple users/teams.
- Ensure resource/object names are unique within a Namespace, but not across different Namespaces.

**Listing Namespaces**:
- Command: `kubectl get namespaces`
- Example output:
```
NAME STATUS AGE
default Active 11h
kube-node-lease Active 11h
kube-public Active 11h
kube-system Active 11h
```

**Default Namespaces (Created by k8s)**:
- <mark style="background: #ADCCFFA6;">kube-system</mark>: Contains objects created by the Kubernetes system, primarily control plane agents.
- <mark style="background: #ADCCFFA6;">default</mark>: Holds objects/resources created by administrators/developers; used by default unless another Namespace is specified.
- <mark style="background: #ADCCFFA6;">kube-public</mark>: Unsecured, readable by anyone, used for exposing non-sensitive cluster information.
- <mark style="background: #ADCCFFA6;">kube-node-lease</mark>: Stores node lease objects for node heartbeat data.

To create a namespace: `kubectl create namespace <name>`

### Pods
![[Pasted image 20250825014628.png]]
- **Definition**:
    - Smallest Kubernetes workload object; unit of deployment.
    - Logical collection of one or more containers, scheduled together, sharing:
        - Network namespace (single Pod IP).
        - External storage (volumes) and dependencies.

- **Characteristics**:
    - Ephemeral; rely on controllers (e.g., Deployments, ReplicaSets, DaemonSets, Jobs) for replication, fault tolerance, self-healing.
    - Pod specification nested in controller’s Pod Template.

- **Pod Definition (YAML Example)**:
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx-pod
      labels:
        run: nginx-pod
    spec:
      containers:
      - name: nginx-pod
        image: nginx:1.22.1
        ports:
        - containerPort: 80
    ```
    - **Fields**:
        - `apiVersion: v1`: Pod API version.
        - `kind: Pod`: Object type.
        - `metadata`: Name and optional labels/annotations.
        - `spec`: Defines desired state (PodSpec), including container image and ports.
    - Kubelet schedules and runs containers using node’s container runtime.

- **Creating Pods**:
    - **Declarative**: Use YAML/JSON file with `kubectl create -f <file>` or `kubectl apply -f <file>`.
    - **Imperative**: `kubectl run nginx-pod --image=nginx:1.22.1 --port=80`.
    - **Generate Definition**:
        - YAML: `kubectl run nginx-pod --image=nginx:1.22.1 --port=80 --dry-run=client -o yaml > nginx-pod.yaml`.
        - JSON: `kubectl run nginx-pod --image=nginx:1.22.1 --port=80 --dry-run=client -o json > nginx-pod.json`.
- **Pod Operations**:
    - Apply: `kubectl apply -f nginx-pod.yaml`.
    - List: `kubectl get pods`.
    - Inspect: `kubectl get pod nginx-pod -o yaml/json` or `kubectl describe pod nginx-pod`.
    - Delete: `kubectl delete pod nginx-pod`.

### Labels
[Labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) are key-value pairs attached to Kubernetes objects such as Pods, ReplicaSets, Nodes, Namespaces and Persistent Volumes. Labels are used to organize and select a subset of objects, based on the requirements in place. Many objects can have the same Label(s). Labels do not provide uniqueness to objects. Controllers use Labels to logically group together decoupled objects, rather than using objects' names or IDs.
![[Pasted image 20250825015829.png]]

Controllers, or operators, and Services, use [label selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors) to select a subset of objects. Kubernetes supports two types of Selectors:

- **Equality-Based Selectors**:
    - Filter objects based on **label keys and values** using:
        - = or == (equals, interchangeable).
        - != (not equals).
    - Example: env=dev or env==dev selects objects with the label env set to dev.
        
- **Set-Based Selectors**:
    - Filter objects based on a **set of values** or **existence of keys** using:
        - in: Selects objects where the label value is in a set (e.g., env in (dev,qa) selects objects with env set to dev or qa).
        - notin: Selects objects where the label value is not in a set.
        - Exists: Selects objects with a specific label key (e.g., app selects objects with the app key).
        - Does not exist: Selects objects without a specific label key (e.g., !app selects objects without the app key).

### ReplicaSets
A [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) is, in part, the next-generation ReplicationController, as it implements the replication and self-healing aspects of the ReplicationController. ReplicaSets support both equality- and set-based Selectors, whereas ReplicationControllers only support equality-based Selectors.
![[Pasted image 20250825020806.png]]

Example ReplicaSet object's definition manifest of a replicaset (redis-rs.yaml):
```yaml
apiVersion: apps/v1  
kind: ReplicaSet  
metadata:  
  name: frontend  
  labels:  
    app: guestbook  
    tier: frontend  
spec:  
  replicas: 3  
  selector:  
    matchLabels:  
      app: guestbook  
  template:  
    metadata:  
      labels:  
        app: guestbook  
    spec:  
      containers:  
      - name: php-redis  
        image: gcr.io/google_samples/gb-frontend:v3**
```


```sh
# Create the above replicaset
kubectl apply -f redis-rs.yaml

# Get ReplicaSets and Pods
kubectl get rs
kubectl get replicatsets
kubectl get rs,po

kubectl scale rs frontend --replicas=4

kubectl get rs frontend -o yaml
kubectl get rs frontend -o json
kubectl get rs,po -o wide

kubectl describe rs frontend

kubectl exec -it <pod-name> -c <container-name> -- <command>

kubectl delete rs frontend

```

ReplicaSets can be used independently as Pod controllers but they only offer a limited set of features. A set of complementary features are provided by Deployments, the recommended controllers for the orchestration of Pods. Deployments manage the creation, deletion, and updates of Pods. A Deployment automatically creates a ReplicaSet, which then creates a Pod. There is no need to manage ReplicaSets and Pods separately, the Deployment will manage them on our behalf.


### Deployments

[Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) objects provide declarative updates to Pods and ReplicaSets. The DeploymentController is part of the control plane node's controller manager, and as a controller it also ensures that the current state always matches the desired state of our running containerized application. It allows for seamless application updates and rollbacks, known as the default _RollingUpdate_ strategy, through **rollouts** and **rollbacks**, and it directly manages its ReplicaSets for application scaling. It also supports a disruptive, less popular update strategy, known as _Recreate_.

Below is an example of a Deployment object's definition manifest in YAML format. This represents the declarative method to define an object, and can serve as a template for a much more complex Deployment definition manifest if desired:

```yaml
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  name: nginx-deployment  
  labels:  
    app: nginx-deployment  
spec:  
  replicas: 3  
  selector:  
    matchLabels:  
      app: nginx-deployment  
  template:  
    metadata:  
      labels:  
        app: nginx-deployment  
    spec:  
      containers:  
      - name: nginx  
        image: nginx:1.20.2  
        ports:  
        - containerPort: 80
```

To come up the above template, we can use the following <mark style="background: #BBFABBA6;">imperative</mark> command `create`:
```sh
kubectl create deployment nginx-deployment \  
--image=nginx:1.20.2 --port=80 --replicas=3 \  
--dry-run=client -o yaml > nginx-deploy.yaml
```

And then `apply`the generated yaml using the following <mark style="background: #FF5582A6;">declarative</mark> method:
```sh
# Apply the deployment file
kubectl apply -f nginx-deploy.yaml

# Display nginx-deploy related resources
kubectl get deploy,rs,po -l app=nginx-deployment
```

Be aware of **Configuration Drift !!!!**

Once the Deployment object is created, the Kubernetes system attaches the **status** field to the object and populates it with all necessary status fields.

In the following example, a new **Deployment** creates **ReplicaSet** **A** which then creates **3 Pods**, with each Pod Template configured to run one **nginx:1.20.2** container image. In this case, the **ReplicaSet A** is associated with **nginx:1.20.2** representing a state of the **Deployment**. This particular state is recorded as **Revision 1**.
![[Pasted image 20250825155900.png]]

### Deployment -  Rolling Update

Let's say we want to update the container image from **nginx:1.20.2** to **nginx:1.21.5**

A **rolling update** is triggered when we update specific properties of the Pod Template for a deployment. While planned changes such as updating the container image, container port, volumes, and mounts would trigger a new Revision, other operations that are dynamic in nature, like scaling or labeling the deployment, do not trigger a rolling update, thus do not change the Revision number.

Once the rolling update has completed, the **Deployment** will show both **ReplicaSets A** and **B**, where **A** is scaled to 0 (zero) Pods, and **B** is scaled to 3 Pods. This is how the Deployment records its prior state configuration settings, as **Revisions**.

**Deployment (ReplicaSet B Created)**
![[Pasted image 20250825161747.png]]


Once **ReplicaSet B** and its **3 Pods** versioned **1.21.5** are ready, the **Deployment** starts actively managing them. However, the Deployment keeps its prior configuration states saved as Revisions which play a key factor in the **<mark style="background: #FF5582A6;">rollback</mark>** capability of the Deployment - returning to a prior known configuration state. In our example, if the performance of the new **nginx:1.21.5** is not satisfactory, the Deployment can be rolled back to a prior Revision, in this case from **Revision 2** back to **Revision 1** running **nginx:1.20.2** once again.

**Deployment Points to ReplicaSet B**
![[Pasted image 20250825161821.png]]

```sh

# Check rollout history
kubectl rollout history deploy <name>
# Show details about a specific revision
kubectl rollout history deploy <name> --revision=1

# Update image (note the container name is defined in deployments.yaml)
kubectl set image deploy <name> CONTAINER_NAME_1=IMAGE_1
# Example rollout update
kubectl set image deploy nginx-deployment nginx=nginx:1.23
# Alternative rollout update
kubectl edit deployment/nginx-deployment


# Example rollback
kubectl rollout undo deploy nginx-deployment --to-revision=1
```


Notice how the original ReplicaSet is remained (scaled down to 0) so that we can rollback using its' state!!!
![[Screenshot 2025-08-25 at 4.36.37 PM.png]]



After rolling back to revision 1, if you check the new rollout history you will only see revision 2 and 3. 

3 represent the current state (which was 1) and 2 remains.

![[Screenshot 2025-08-25 at 4.44.47 PM.png]]


### Daemon Sets

A DaemonSet defines Pods that provide node-local facilities. These might be fundamental to the operation of your cluster, such as a networking helper tool, or be part of an add-on.

<mark style="background: #ADCCFFA6;">A DaemonSet ensures that all (or some) Nodes run a copy of a Pod</mark>. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Deleting a DaemonSet will clean up the Pods it created.

**fluentd-ds.yaml**
```yaml
apiVersion: apps/v1  
kind: DaemonSet  
metadata:  
  name: fluentd-agent  
  namespace: default  
  labels:  
    k8s-app: fluentd-agent  
spec:  
  selector:  
    matchLabels:  
      k8s-app: fluentd-agent  
  template:  
    metadata:  
      labels:  
        k8s-app: fluentd-agent  
    spec:  
      containers:  
      - name: fluentd  
        image: quay.io/fluentd_elasticsearch/fluentd:v4.5.2
```