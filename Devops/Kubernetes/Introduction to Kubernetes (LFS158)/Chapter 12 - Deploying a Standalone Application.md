
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

## Liveness

If a container in the Pod has been running successfully for a while, but the application running inside this container suddenly stopped responding to our requests, then that container is no longer useful to us. This kind of situation can occur, for example, due to application deadlock or memory pressure. In such a case, it is recommended to restart the container to make the application available.

<mark style="background: #ADCCFFA6;">Rather than restarting it manually, we can use a Liveness Probe. Liveness Probe checks on an application's health, and if the health check fails, **kubelet** restarts the affected container automatically.</mark>

Liveness Probes can be set by defining:

- Liveness command
- Liveness HTTP request
- TCP Liveness probe
- gRPC Liveness probe

## Liveness Command


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