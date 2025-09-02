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