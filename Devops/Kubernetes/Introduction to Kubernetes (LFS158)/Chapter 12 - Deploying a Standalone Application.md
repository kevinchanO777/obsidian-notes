
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

## [Liveness, Readiness, and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

While containerized applications are scheduled to run in pods on nodes across our cluster, at times the applications may become unresponsive or may be delayed during startup. Implementing [Liveness and Readiness Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) allows the **kubelet** to control the health of the application running inside a Pod's container and force a container restart of an unresponsive application.

In the next few sections, we will discuss probes in more detail.

## Liveness Probes

If a container in the Pod has been running successfully for a while, but the application running inside this container suddenly stopped responding to our requests, then that container is no longer useful to us. This kind of situation can occur, for example, due to application deadlock or memory pressure. In such a case, it is recommended to restart the container to make the application available.

<mark style="background: #ADCCFFA6;">Rather than restarting it manually, we can use a Liveness Probe. Liveness Probe checks on an application's health, and if the health check fails, **kubelet** restarts the affected container automatically.</mark>

Liveness Probes can be set by defining:

- Liveness command
- Liveness HTTP request
- TCP Liveness probe
- gRPC Liveness probe

###  Liveness - Command

In the following example, the [**liveness**](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-a-liveness-command) command is checking the existence of a file **/tmp/healthy**:


```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 15
      failureThreshold: 1
      periodSeconds: 5
```


The existence of the **/tmp/healthy** file is configured to be checked every 5 seconds using the **periodSeconds** parameter. The **initialDelaySeconds** parameter requests the kubelet to wait for 15 seconds before the first probe. When running the command line argument to the container, we will first create the **/tmp/healthy** file, and then we will remove it after 30 seconds. The removal of the file would trigger a probe failure, while the **failureThreshold** parameter set to **1** instructs kubelet to declare the container unhealthy after a single probe failure and trigger a container restart as a result.

A demonstration video covering this method is up next.

### Liveness - HTTP Request

In the following example, the kubelet sends the [**HTTP GET** request](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-a-liveness-http-request) to the **/healthz** endpoint of the application, on port **8080**. If that returns a failure, then the kubelet will restart the affected container; otherwise, it would consider the application to be alive. 

Any code greater than or equal to 200 and less than 400 indicates success. Any other code indicates failure.

```yaml

...
     livenessProbe:
       httpGet:
         path: /healthz
         port: 8080
         httpHeaders:
         - name: X-Custom-Header
           value: Awesome
       initialDelaySeconds: 15
       periodSeconds: 5
...

```

### Liveness - TCP Liveness Probe

[TCP Liveness Probe](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-a-tcp-liveness-probe) With this configuration, the kubelet will attempt to open a socket to your container on the specified port. If it can establish a connection, the container is considered healthy, if it can't it is considered a failure.

```yaml

...
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 5
...

```


### Liveness - gRPC Liveness Probe


The [gRPC Liveness Probe](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-a-grpc-liveness-probe) can be used for applications implementing the [gRPC health checking protocol](https://github.com/grpc/grpc/blob/master/doc/health-checking.md). It requires for a **port** to be defined, and optionally a **service** field may help adapt the probe for liveness or readiness by allowing the use of the same port.

```yaml

...
    livenessProbe:
      grpc:
        port: 2379
      initialDelaySeconds: 10
...

```


## Readiness Probes

Sometimes, while initializing, applications have to meet certain conditions before they become ready to serve traffic. These conditions include ensuring that the dependent service is ready, or acknowledging that a large dataset needs to be loaded, etc. In such cases, we use [Readiness Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-readiness-probes) and wait for a certain condition to occur. Only then, the application can serve traffic.

A Pod with containers that do not report ready status will not receive traffic from Kubernetes Services.

```yaml
...
    readinessProbe:
          exec:
            command:
            - cat
            - /tmp/healthy
          initialDelaySeconds: 5 
          periodSeconds: 5
...
```

Readiness Probes are configured similarly to Liveness Probes. Their configuration fields and options also remain the same. Readiness probes are also defined as Readiness command, Readiness HTTP request, TCP readiness probe, and gRPC readiness probe.

Please review the [Readiness Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-readiness-probes) for more details.


## Startup Probes

The newest member of the Probes family is the [Startup Probe](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-startup-probes). This probe was designed for legacy applications that may need more time to fully initialize and its purpose is to delay the Liveness and Readiness probes, a delay long enough to allow for the application to fully initialize.

```yaml
# This define a named port for TCP and HTTP only.
ports:
- name: liveness-port
  containerPort: 8080

livenessProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 1
  periodSeconds: 10

startupProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 30
  periodSeconds: 10
```

Thanks to the startup probe, the application will have a maximum of 5 minutes (30 * 10 = 300s) to finish its startup. Once the startup probe has succeeded once, the liveness probe takes over to provide a fast response to container deadlocks. If the startup probe never succeeds, the container is killed after 300s and subject to the pod's `restartPolicy`.