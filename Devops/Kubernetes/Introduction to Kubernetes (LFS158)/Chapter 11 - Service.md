```table-of-contents
title: 
style: nestedList # TOC style (nestedList|nestedOrderedList|inlineFirstLevel)
minLevel: 0 # Include headings from the specified level
maxLevel: 0 # Include headings up to the specified level
include: 
exclude: 
includeLinks: true # Make headings clickable
hideWhenEmpty: false # Hide TOC if no headings are found
debugInConsole: false # Print debug info in Obsidian console
```

To access the application, a user or another application needs to connect to a Pod running the target application. As Pods are ephemeral in nature, resources like IP addresses allocated to them cannot be static. Pods could be terminated abruptly or be rescheduled based on existing requirements.

-> IP address of new pod changes
![[Pasted image 20250901182133.png]]

To overcome this situation, Kubernetes provides a higher-level abstraction called _Service_, which logically groups Pods and defines a policy to access them. This grouping is achieved via _Labels_ and _Selectors_. This logical grouping strategy is used by Pod controllers, such as ReplicaSets, Deployments, and even DaemonSets. Below is a Deployment definition manifest for the **frontend** app, to aid with the correlation of Labels, Selectors, and port values between the Deployment controller, its Pod replicas, and the Service definition manifest presented in an upcoming section.

`frontend-deploy.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: frontend
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
    template:
      metadata:
        labels:
          app: frontend
      spec:
        containers:
        - image: <frontend-application-image>
        name: frontend-application
        ports:
        - containerPort: 5000
```


Labels and Selectors use a _key-value_ pair format. In the following graphical representation

| key       | app              |
| --------- | ---------------- |
| **value** | **frontend, db** |


**Grouping of Pods using Labels and Selectors**
![[Pasted image 20250901182231.png]]

Using the selectors app frontend and **app\==db**, we group Pods into two logical sets: one set with 3 Pods, and one set with a single Pod.

We assign a name to the logical grouping, referred to as a Service. The Service name is also registered with the cluster's internal DNS service. In our example, we create two Services, frontend-svc, and db-svc, and they have the app\==frontend and the app\==db Selectors, respectively.

Services can expose single Pods, ReplicaSets, Deployments, DaemonSets, and StatefulSets. When exposing the Pods managed by an operator, the Service's Selector may use the same label(s) as the operator. A clear benefit of a Service is that it watches application Pods for any changes in count and their respective IP addresses while automatically updating the list of corresponding endpoints. Even for a single-replica application, run by a single Pod, the Service is beneficial during self-healing (replacement of a failed Pod) as it immediately directs traffic to the newly deployed healthy Pod.

## Service Object Example

The following is an example of a Service object definition. This represents the declarative method to define an object, and can serve as a template for a much more complex Service definition manifest if desired. By omitting the Service **type** from the definition manifest, we create the default service type, the **ClusterIP** type (the ClusterIP Service type will be covered in an upcoming lesson).

`forntend-svc.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5000
```

To deploy we call:
```bash
kubectl apply -f 'frontend-svc.yaml'

# To expose the Deployment Pods
kubectl expose deploy frontend --name=frontend-svc --port=80 --target-port=5000
```

The **expose** command parses the referenced Deployment object to extract valuable pairing details such as Name, Label, Selector, and containerPort to populate these values in the Service object. However, especially in cases when the Service **port** and Service **targetPort** values are expected to be distinct (**80** and **5000** respectively), it is best to explicitly supply these values with the **expose** command. In addition, we decided to change the name of the Service with the **name** option (the default behavior is for the Service object to inherit the exposed Deployment’s name **frontend**).

```bash
kubectl get service,endpoints frontend-svc
# or
**kubectl get svc,ep frontend-svc**
```

![[Pasted image 20250901202307.png]]

Now ssh into minikube and try `curl <service_ip>:<service_port>` which will sent a get request to one of the pod at the defined exposed port i.e. 5000 


This example map service port 80 to pod port 80 which the container is running nginx. You see how k8s automatically load balance our requests.
![[Pasted image 20250901203415.png]]


## kube-proxy

![[Pasted image 20250902143405.png]]

Each cluster node runs a daemon called [kube-proxy](https://kubernetes.io/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies), a node agent that watches the API server on the control plane node for the addition, updates, and removal of Services and endpoints. **kube-proxy** is responsible for implementing the Service configuration on behalf of an administrator or developer, in order to enable traffic routing to an exposed application running in Pods. In the example below, for each new Service, on each node, **kube-proxy** configures **iptables** rules to capture the traffic for its **ClusterIP** and forwards it to one of the Service's endpoints. Therefore any node can receive the external traffic and then route it internally in the cluster based on the **iptables** rules. When the Service is removed, **kube-proxy** removes the corresponding **iptables** rules on all nodes as well.

Just as the **kube-proxy** node agent runs redundantly on each cluster node, the **iptables** are populated in a redundant fashion by their respective node agents so that each **iptables** instance stores complete routing rules for the entire cluster. This helps with the Service objects implementation to reproduce a distributed load balancing mechanism.


## Traffic Policies

The kube-proxy node agent together with the iptables implement the load-balancing mechanism of the Service when traffic is being routed to the application Endpoints. Due to restricting characteristics of the iptables this load-balancing is random by default. This means that the Endpoint Pod to receive the request forwarded by the Service will be randomly selected out of many replicas. This mechanism does not guarantee that the selected receiving Pod is the closest or even on the same node as the requester, therefore not the most efficient mechanism. Since this is the iptables supported load-balancing mechanism, if we desire better outcomes, we would need to take advantage of traffic policies.

[Traffic policies](https://kubernetes.io/docs/reference/networking/virtual-ips/#traffic-policies) allow users to instruct the kube-proxy on the context of the traffic routing. The two options are Cluster and Local:

- The _Cluster_ option allows kube-proxy to target all ready Endpoints of the Service in the load-balancing process. This is the default behavior of the Service even when the traffic policy property is not explicitly declared.
- The _Local_ option, however, isolates the load-balancing process to only include the Endpoints of the Service located on the same node as the requester Pod, or perhaps the node that captured inbound external traffic on its NodePort (the NodePort Service type will be covered in an upcoming lesson). While this sounds like an ideal option, it does have a shortcoming - if the Service does not have a ready Endpoint on the node where the requester Pod is running, the Service will not route the request to Endpoints on other nodes to satisfy the request because it will be dropped by kube-proxy.

Both the Cluster and Local options are available for requests generated internally from within the cluster, or externally from applications and clients running outside the cluster. The Service definition manifest below defines both internal and external Local traffic policies. Both are optional settings, and can be used independent of each other, where one can be defined without the other (either internal or external policy).

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  internalTrafficPolicy: Local
  externalTrafficPolicy: Local
```


## Service Discovery

As Services are the primary mode of communication between containerized applications managed by Kubernetes, it is helpful to be able to discover them at runtime. Kubernetes supports two methods for discovering Services.

1. Environment variables

As soon as the Pod starts on any worker node, the **kubelet** daemon running on that node adds a set of environment variables in the Pod for all active Services. For example, if we have an active Service called **redis-master**, which exposes port **6379**, and its **ClusterIP** is **172.17.0.6**, then, on a newly created Pod, we can see the following environment variables: `kubectl exec -it <pod> -- printenv`

**REDIS_MASTER_SERVICE_HOST=172.17.0.6**  
**REDIS_MASTER_SERVICE_PORT=6379**  
**REDIS_MASTER_PORT=tcp://172.17.0.6:6379**  
**REDIS_MASTER_PORT_6379_TCP=tcp://172.17.0.6:6379**  
**REDIS_MASTER_PORT_6379_TCP_PROTO=tcp**  
**REDIS_MASTER_PORT_6379_TCP_PORT=6379**  
**REDIS_MASTER_PORT_6379_TCP_ADDR=172.17.0.6**

<mark style="background: #FF5582A6;">With this solution, we need to be careful while ordering our Services, as the Pods will not have the environment variables set for Services which are created after the Pods are created.
</mark>

2. DNS

Kubernetes has an add-on for [DNS](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/), which creates a DNS record for each Service and its format is **my-svc.my-namespace.svc.cluster.local**. Services within the same Namespace find other Services just by their names. If we add a Service **redis-master** in ;**my-ns** Namespace, all Pods in the same **my-ns** Namespace lookup the Service just by its name, **redis-master**. Pods from other Namespaces, such as **test-ns**, lookup the same Service by adding the respective Namespace as a suffix, such as **redis-master.my-ns** or providing the FQDN of the service as **redis-master.my-ns.svc.cluster.local**.

This is the most common and highly recommended solution. For example, in the previous section's image, we have seen that an internal DNS is configured, which maps our Services **frontend-svc** and **db-svc** to **172.17.0.4** and **172.17.0.5** IP addresses respectively.

If we had a client application accessing the frontend application, the client would only need to “know” the frontend application’s Service name and port, which are frontend-svc and port 80 respectively. From a client application Pod we could possibly run the following command, allowing for the cluster internal name resolution and the kube-proxy to guide the client’s request to a frontend Pod:


```bash
kubectl exec client-app-pod-name -c client-container-name -- /bin/sh -c curl -s frontend-svc:80
```

## ServiceType

While defining a Service, we can also choose its access scope. We can decide whether the Service:

- Is only accessible within the cluster.
- Is accessible from within the cluster and the external world.
- Maps to an entity which resides either inside or outside the cluster.

Access scope is decided by **ServiceType** property, defined when creating the Service.

![[Pasted image 20250902150456.png]]



### ServiceType - ClusterIP

**ClusterIP** is the default _[ServiceType](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)_. A Service receives a Virtual IP address, known as its ClusterIP. This Virtual IP address is used for communicating with the Service and is accessible <mark style="background: #ADCCFFA6;">only</mark> from within the cluster. The **frontend-svc** Service definition manifest now includes an explicit **type** for ClusterIP. If omitted, the default ClusterIP service type is set up:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5000
  type: ClusterIP
```


### ServiceType - [Headless Services](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services)

For headless Services, a cluster IP is not allocated, kube-proxy does not handle these Services, and there is no load balancing or proxying done by the platform for them.

A headless Service allows a client to connect to whichever Pod it prefers, directly. Services that are headless don't configure routes and packet forwarding using [virtual IP addresses and proxies](https://kubernetes.io/docs/reference/networking/virtual-ips/); instead, headless Services report the endpoint IP addresses of the individual pods via internal DNS records, served through the cluster's [DNS service](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/). To define a headless Service, you make a Service with `.spec.type` set to ClusterIP (which is also the default for `type`), and you additionally set `.spec.clusterIP` to None.

The string value None is a special case and is not the same as leaving the `.spec.clusterIP` field unset.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5000
  type: None
```

```bash
# Optional
apt install dnsutils

# Inside a pod
nslookup frontend-svc

# Should return all pod IPs

```
### ServiceType - NodePort

With the [**NodePort**](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport) _ServiceType_, in addition to a ClusterIP, a high-port, dynamically picked from the default range **30000-32767**, is mapped to the respective Service, from all the worker nodes. For example, if the mapped NodePort is **32233** for the service **frontend-svc**, then, if we connect to any worker node on port **32233**, the node would redirect all the traffic to the assigned ClusterIP - **172.17.0.4**. If we prefer a specific high-port number instead, then we can assign that high-port number to the NodePort from the default range when creating the Service.

![[Pasted image 20250902150846.png]]

The **NodePort** _ServiceType_ is useful when we want to make our Services accessible from the external world.<mark style="background: #ADCCFFA6;"> The end-user connects to any worker node on the specified high-port, which proxies the request internally to the ClusterIP of the Service</mark>, then the request is forwarded to the applications running inside the cluster. Let's not forget that the Service is load balancing such requests, and only forwards the request to one of the Pods running the desired application. <mark style="background: #ADCCFFA6;">To manage access to multiple application Services from the external world, administrators can configure a reverse proxy - an ingress, and define rules that target specific Services within the cluster.</mark>

The NodePort type has to be explicitly declared in the Service definition manifest or with the imperative methods explored in an earlier lesson - the **expose** and **create service** commands. Declaring the **nodePort** value **32233** is optional, ensuring there is no conflict. We are reusing the earlier definition and commands updated for the NodePort **type** and declaring the **nodePort** value where supported:

```bash
kubectl expose deploy frontend --name=frontend-svc \
--port=80 --target-port=5000 --type=NodePort 

kubectl create service nodeport frontend-svc \
--tcp=80:5000 --node-port=32233
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5000
    nodePort: 32233 # Optional
  type: NodePort
```

### ServiceType - LoadBalancer

With the **[LoadBalancer](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer)** _ServiceType_:

- NodePort and ClusterIP are automatically created, and the external load balancer will route to them.
- The Service is exposed at a static port on each worker node.
- The Service is exposed externally using the underlying cloud provider's load balancer feature.

![[Pasted image 20250902154428.png]]

The **LoadBalancer** _ServiceType_ will only work if the underlying infrastructure supports the automatic creation of Load Balancers and have the respective support in Kubernetes, as is the case with the Google Cloud Platform and AWS. <mark style="background: #FF5582A6;">If no such feature is configured, the LoadBalancer IP address field is not populated, it remains in Pending state, but the Service will still work as a typical NodePort type Service.</mark>


### Service Type - ExternalName

**[ExternalName](https://kubernetes.io/docs/concepts/services-networking/service/#externalname)** is a special _ServiceType_ that has no Selectors and does not define any endpoints. When accessed within the cluster, it returns a **CNAME** record of an externally configured Service.

<mark style="background: #ADCCFFA6;">The primary use case of this _ServiceType_ is to make externally configured Services like **my-database.example.com** available to applications inside the cluster.</mark> If the externally defined Service resides within the same Namespace, using just the name **my-database** would make it available to other applications and Services within that same Namespace.

Services of type ExternalName map a Service to a DNS name, not to a typical selector such as `my-service` or `cassandra`. You specify these Services with the `spec.externalName` parameter.

This Service definition, for example, maps the `my-service` Service in the `prod` namespace to `my.database.example.com`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```


### Service Type - ExternalIP

If there are external IPs that route to one or more cluster nodes, Kubernetes Services can be exposed on those `externalIPs`. When network traffic arrives into the cluster, with the external IP (as destination IP) and the port matching that Service, rules and routes that Kubernetes has configured ensure that the traffic is routed to one of the endpoints for that Service.

When you define a Service, you can specify `externalIPs` for any [service type](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types). In the example below, the Service named `"my-service"` can be accessed by clients using TCP, on `"198.51.100.32:80"` (calculated from `.spec.externalIPs[]` and `.spec.ports[].port`).

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 49152
  externalIPs:
    - 198.51.100.32
```


>Kubernetes does not manage allocation of `externalIPs`; these are the responsibility of the cluster administrator.


## [Multi-Port Services](https://kubernetes.io/docs/concepts/services-networking/service/#multi-port-services)

For some Services, you need to expose more than one port. Kubernetes lets you configure multiple port definitions on a Service object. When using multiple ports for a Service, you must give all of your ports names so that these are unambiguous. For example:

A multi-port Service manifest is provided below:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
    - name: https
      protocol: TCP
      port: 443
      targetPort: 9377
```


> [!info] Note
> As with Kubernetes [names](https://kubernetes.io/docs/concepts/overview/working-with-objects/names) in general, names for ports must only contain lowercase alphanumeric characters and `-`. Port names must also start and end with an alphanumeric character.
> 
> For example, the names `123-abc` and `web` are valid, but `123_abc` and `-web` are not.


### Port Forwarding

Another application exposure mechanism in Kubernetes is port forwarding. In Kubernetes the port forwarding feature allows users to easily forward a local port to an application port. Thanks to its flexibility, the application port can be a Pod container port, a Service port, and even a Deployment container port (from its Pod template). This allows users to test and debug their application running in a remote cluster by targeting a port on their local workstation (either **[http://localhost:port](http://localhost:port/)** or **[http://127.0.0.1:port](http://127.0.0.1:port/)**), a solution for remote cloud clusters or virtualized on premises clusters.

Port forwarding can be utilized as an alternative to the NodePort Service type because it does not require knowledge of the public IP address of the Kubernetes Node. As long as there are no firewalls blocking access to the desired local workstation port, such as 8080 in the examples below, the port forwarding method can quickly allow access to the application running in the cluster.

Example:

```bash
kubectl port-forward deploy/frontend 8080:5000 

kubectl port-forward frontend-77cbdf6f79-qsdts 8080:5000 

kubectl port-forward svc/frontend-svc 8080:80
```

All three commands forward port 8080 of the local workstation to the container port 5000 of the Deployment and Pod respectively, and to the Service port 80. While the Pod resource type is implicit, therefore optional and can be omitted, the Deployment and Service resource types are required to be explicitly supplied in the presented syntax.


Or using k9s:
```bash
# Open k9s
k9s

# Go to pod tab
:pod 

# Sift + f to start port forwarding
<shift-f>
```

The port forwarding process will close after you quit k9s 
![[Pasted image 20250902160926.png]]
