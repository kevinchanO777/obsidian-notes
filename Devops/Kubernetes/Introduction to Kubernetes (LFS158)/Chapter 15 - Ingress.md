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
## [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)

Make your HTTP (or HTTPS) network service available using a protocol-aware configuration mechanism, that understands web concepts like URIs, hostnames, paths, and more. The Ingress concept lets you map traffic to different backends based on rules you define via the Kubernetes API.

With Services, routing rules are associated with a given Service. They exist for as long as the Service exists, and there are many rules because there are many Services in the cluster. If we can somehow decouple the routing rules from the application and centralize the rules management, we can then update our application without worrying about its external access. This can be done using the _Ingress_ resource - a collection of rules that manage inbound connections to cluster Services.

To allow the inbound connection to reach the cluster Services, Ingress configures a Layer 7 HTTP/HTTPS load balancer for Services and provides the following:

- TLS (Transport Layer Security)
- Name-based virtual hosting
- Fanout routing
- Loadbalancing
- Custom rules.


![[Pasted image 20250905164552.png]]


### Name Based Virtual Hosting

With Ingress, users do not connect directly to a Service. Users reach the Ingress endpoint, and, from there, the request is forwarded to the desired Service. You can see an example of a _[Name-Based Virtual Hosting](https://kubernetes.io/docs/concepts/services-networking/ingress/#name-based-virtual-hosting) Ingress_ definition below:

```yaml
apiVersion: networking.k8s.io/v1 
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/service-upstream: "true"
  name: virtual-host-ingress
  namespace: default
spec:
  ingressClassName: nginx
  rules:
  - host: blue.example.com
    http:
      paths:
      - backend:
          service:
            name: webserver-blue-svc
            port:
              number: 80
        path: /
        pathType: ImplementationSpecific
  - host: green.example.com
    http:
      paths:
      - backend:
          service:
            name: webserver-green-svc
            port:
              number: 80
        path: /
        pathType: ImplementationSpecific
```

In the example above, user requests to both **blue.example.com** and **green.example.com** would go to the same Ingress endpoint, and, from there, they would be forwarded to **webserver-blue-svc**, and **webserver-green-svc**, respectively.


**Name-Based Virtual Hosting Ingress**
![[Pasted image 20250905170030.png]]



### Simple Fanout

We can also define _[Fanout](https://kubernetes.io/docs/concepts/services-networking/ingress/#simple-fanout)_ Ingress rules, presented in the example definition and the diagram below, when requests to example.com/blue and example.com/green would be forwarded to **webserver-blue-svc** and **webserver-green-svc**, respectively:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/service-upstream: "true"
  name: fan-out-ingress
  namespace: default
spec:
  ingressClassName: nginx 
  rules:
  - host: example.com
    http:
      paths:
      - path: /blue
        backend:
          service:
            name: webserver-blue-svc
            port:
              number: 80
        pathType: ImplementationSpecific
      - path: /green
        backend:
          service:
            name: webserver-green-svc
            port:
              number: 80
        pathType: ImplementationSpecific
```

**Fanout Ingress**
![[Pasted image 20250905170236.png]]

The Ingress resource does not do any request forwarding by itself, it merely accepts the definitions of traffic routing rules. The ingress is fulfilled by an Ingress Controller, which is a reverse proxy responsible for traffic routing based on rules defined in the Ingress resource.

## Ingress Controller

An [Ingress Controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) is an application watching the Control Plane Node's API server for changes in the Ingress resources and updates the Layer 7 Load Balancer accordingly. An Ingress Controller is also known as Controllers, Ingress Proxy, Service Proxy, Reverse Proxy, etc. Kubernetes supports an array of Ingress Controllers, and, if needed, we can also build our own. [GCE L7 Load Balancer Controller](https://github.com/kubernetes/ingress-gce/blob/master/README.md), [AWS Load Balancer Controller](https://github.com/kubernetes-sigs/aws-load-balancer-controller#readme), and [Nginx Ingress Controller](https://github.com/kubernetes/ingress-nginx/blob/master/README.md) are commonly used Ingress Controllers. Other controllers are [Contour](https://projectcontour.io/), [HAProxy Ingress](https://haproxy-ingress.github.io/), [Istio Ingress](https://istio.io/latest/docs/tasks/traffic-management/ingress/kubernetes-ingress/), [Kong](https://konghq.com/), [Traefik](https://traefik.io/traefik/), etc. In order to ensure that the ingress controller is watching its corresponding ingress resource, the ingress resource definition manifest needs to include an ingress class name, such as **spec.ingressClassName: nginx** and optionally one or several annotations specific to the desired controller, such as **nginx.ingress.kubernetes.io/service-upstream: "true"** (for an nginx ingress controller).

Starting the Ingress Controller in Minikube is extremely simple. Minikube ships with the [Nginx Ingress Controller](https://www.nginx.com/products/nginx/kubernetes-ingress-controller) set up as an add-on, disabled by default. It can be easily enabled by running the following command:

```bash
minikube addons enable ingress
```