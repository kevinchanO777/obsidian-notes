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
## Volumes

As we know, containers running in Pods are ephemeral in nature. All data stored inside a container is deleted if the container crashes. However, the **kubelet** will restart it with a clean slate, which means that it will not have any of the old data.

To overcome this problem, Kubernetes uses ephemeral [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/), storage abstractions that allow various storage technologies to be used by Kubernetes and offered to containers in Pods as storage media. An ephemeral Volume is essentially a mount point on the container's file system backed by a storage medium. The storage medium, content and access mode are determined by the Volume Type.

<mark style="background: #FF5582A6;">In Kubernetes, an ephemeral Volume is linked to a Pod and can be shared among the containers of that Pod.</mark> Although the ephemeral Volume has the same life span as the Pod, meaning that it is deleted together with the Pod, the ephemeral Volume outlives the containers of the Pod - this allows data to be preserved across container restarts.

![[Pasted image 20250902200814.png]]

## Container Storage Interface (CSI)

Container orchestrators like Kubernetes, Mesos, Docker or Cloud Foundry used to have their own methods of managing external storage using Volumes. For storage vendors, it was challenging to manage different Volume plugins for different orchestrators. A maintainability challenge for Kubernetes as well, it involved in-tree storage plugins integrated into the orchestrator's source code. Storage vendors and community members from different orchestrators started working together to standardize the Volume interface - a volume plugin built using a standardized [Container Storage Interface](https://kubernetes.io/docs/concepts/storage/volumes/#csi) (CSI) designed to work on different container orchestrators with a variety of storage providers. Explore the [CSI specifications](https://github.com/container-storage-interface/spec/blob/master/spec.md) for more details.

Between Kubernetes releases v1.9 and v1.13 CSI matured from alpha to [stable support](https://kubernetes.io/blog/2019/01/15/container-storage-interface-ga/), which makes installing new CSI-compliant Volume drivers very easy. CSI allows third-party storage providers to [develop solutions](https://kubernetes-csi.github.io/docs/) without the need to add them into the core Kubernetes codebase. These solutions are CSI drivers installed only when required by cluster administrators.


## Volume Types

A directory which is mounted inside a Pod is backed by the underlying [Volume Type](https://kubernetes.io/docs/concepts/storage/volumes/#volume-types). A Volume Type decides the properties of the directory, like size, content, default access modes, etc. Some examples of Volume Types that support ephemeral Volumes are:

### [emptyDir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir)

For a Pod that defines an `emptyDir` volume, the volume is created when the Pod is assigned to a node. As the name says, the `emptyDir` volume is initially empty. All containers in the Pod can read and write the same files in the `emptyDir` volume, though that volume can be mounted at the same or different paths in each container. When a Pod is removed from a node for any reason, the data in the `emptyDir` is deleted permanently.


Some uses for an `emptyDir` are:

- scratch space, such as for a disk-based merge sort
- checkpointing a long computation for recovery from crashes
- holding files that a content-manager container fetches while a webserver container serves the data

The `emptyDir.medium` field controls where `emptyDir` volumes are stored. By default `emptyDir` volumes are stored on whatever medium that backs the node such as disk, SSD, or network storage, depending on your environment. If you set the `emptyDir.medium` field to `"Memory"`, Kubernetes mounts a tmpfs (RAM-backed filesystem) for you instead. While tmpfs is very fast, be aware that, unlike disks, files you write count against the memory limit of the container that wrote them.

A size limit can be specified for the default medium, which limits the capacity of the `emptyDir` volume. The storage is allocated from [node ephemeral storage](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#setting-requests-and-limits-for-local-ephemeral-storage). If that is filled up from another source (for example, log files or image overlays), the `emptyDir` may run out of capacity before this limit. If no size is specified, memory-backed volumes are sized to node allocatable memory.


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: registry.k8s.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir:
      sizeLimit: 500Mi
```



### [configMap](https://kubernetes.io/docs/concepts/configuration/configmap/)

A ConfigMap is an API object used to store non-confidential data in key-value pairs. [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a [volume](https://kubernetes.io/docs/concepts/storage/volumes/).

A ConfigMap allows you to decouple environment-specific configuration from your [container images](https://kubernetes.io/docs/reference/glossary/?all=true#term-image), so that your applications are easily portable.


You can learn more details about Volume Types from the [documentation](https://kubernetes.io/docs/concepts/storage/volumes/). However, do not be alarmed by the “deprecated” and “removed” notices. They have been added as means of tracking the original in-tree plugins which eventually migrated to the CSI driver implementation. Kubernetes native plugins do not show such a notice.