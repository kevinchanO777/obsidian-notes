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


## [Secret](https://kubernetes.io/docs/concepts/configuration/secret/)



Volume Demo:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: blue-app
  name: blue-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: blue-app
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: blue-app
        type: canary
    spec:
      volumes:
        - name: host-volume
          hostPath:
            path: /home/docker/blue-shared-volume
      containers:
        - image: nginx
          name: nginx
          ports:
            - containerPort: 80
          volumeMounts:
            - mountPath: /usr/share/nginx/html
              name: host-volume
        - image: debian
          name: debian
          volumeMounts:
            - mountPath: /host-vol
              name: host-volume
          command:
            [
              "/bin/sh",
              "-c",
              "echo Welcome to BLUE App! > /host-vol/index.html ; sleep infinity",
            ]
status: {}

```


## PersistentVolumes

> **PV is NOT namespaced**

In a typical IT environment, storage is managed by the storage/system administrators. The end user will just receive instructions to use the storage but is not involved with the underlying storage management.

In the containerized world, we would like to follow similar rules, but it becomes challenging, given the many Volume Types we have seen earlier. Kubernetes resolves this problem with the [PersistentVolume (PV)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) subsystem, which provides APIs for users and administrators to manage and consume persistent storage.<mark style="background: #FF5582A6;"> To manage the Volume, it uses the PersistentVolume API resource type, and to consume it, it uses the PersistentVolumeClaim API resource type.</mark>

A Persistent Volume is a storage abstraction backed by several storage technologies, which could be local to the host where the Pod is deployed with its application container(s), network attached storage, cloud storage, or a distributed storage solution. A Persistent Volume is statically provisioned by the cluster administrator.


PersistentVolumes can be [dynamically](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/) provisioned based on the StorageClass resource. A [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/) contains predefined provisioners and parameters to create a PersistentVolume. Using PersistentVolumeClaims, a user sends the request for dynamic PV creation, which gets wired to the StorageClass resource.

![[Pasted image 20250902205043.png]]

Some of the Volume Types that support managing storage using PersistentVolumes are:

- GCEPersistentDisk
- AWSElasticBlockStore
- AzureFile
- AzureDisk
- CephFS
- NFS
- iSCSI.

For a complete list, as well as more details, you can check out the [types of Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes). The Persistent Volume types use the same CSI driver implementations as ephemeral Volumes.


## PersistentVolumeClaims

A [PersistentVolumeClaim (PVC)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) is a request for storage by a user. Users request for PersistentVolume resources based on storage class, [access mode](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes), size, and optionally volume mode. 

There are four access modes: 

- ReadWriteOnce -> read-write by a single node
- ReadOnlyMany -> read-only by many nodes
- ReadWriteMany -> read-write by many nodes
- ReadWriteOncePod -> read-write by a single pod

The optional volume modes, filesystem or block device, allow volumes to be mounted into a pod's directory or as a raw block device respectively. By design Kubernetes does not support object storage, but it can be implemented with the help of custom resource types. Once a suitable PersistentVolume is found, it is bound to a PersistentVolumeClaim.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow
  selector:
    matchLabels:
      release: "stable"
    matchExpressions:
      - {key: environment, operator: In, values: [dev]}
```


### [Claims As Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes)

Pods access storage by using the claim as a volume. Claims must exist in the same namespace as the Pod using the claim. The cluster finds the claim in the Pod's namespace and uses it to get the PersistentVolume backing the claim. The volume is then mounted to the host and into the Pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```


**PersistentVolumeClaim**
![[Pasted image 20250902205359.png]]

**PersistentVolumeClaim Used In a Pod**
![[Pasted image 20250902223134.png]]
Once a user finishes its work, the attached PersistentVolumes can be released. The underlying PersistentVolumes can then be _reclaimed_ (for an admin to verify and/or aggregate data), _deleted_ (both data and volume are deleted), or _recycled_ for future usage (only data is deleted), based on the configured **persistentVolumeReclaimPolicy** property.

To learn more, you can check out the [PersistentVolumeClaims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims).


### [Types of PVC](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes)

PersistentVolume types are implemented as plugins. Kubernetes currently supports the following plugins:

- [`csi`](https://kubernetes.io/docs/concepts/storage/volumes/#csi) - Container Storage Interface (CSI)
- [`fc`](https://kubernetes.io/docs/concepts/storage/volumes/#fc) - Fibre Channel (FC) storage
- [`hostPath`](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath) - HostPath volume (for single node testing only; WILL NOT WORK in a multi-node cluster; consider using `local` volume instead)
- [`iscsi`](https://kubernetes.io/docs/concepts/storage/volumes/#iscsi) - iSCSI (SCSI over IP) storage
- [`local`](https://kubernetes.io/docs/concepts/storage/volumes/#local) - local storage devices mounted on nodes.
- [`nfs`](https://kubernetes.io/docs/concepts/storage/volumes/#nfs) - Network File System (NFS) storage


PersistentVolume Demo: 
1. https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/
2. https://medium.com/@muppedaanvesh/a-hand-on-guide-to-kubernetes-volumes-%EF%B8%8F-b59d4d4e347f

## [Storage Class](https://kubernetes.io/docs/concepts/storage/storage-classes/)

<mark style="background: #FF5582A6;">SC</mark> provisions <mark style="background: #FF5582A6;">PV</mark> **dynamically** when <mark style="background: #FF5582A6;">PVC</mark> claims it.

Example:
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: low-latency
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"
provisioner: kubernetes.io/aws-ebs # Example
reclaimPolicy: Retain # default value is Delete
allowVolumeExpansion: true
mountOptions:
  - discard # this might enable UNMAP / TRIM at the block storage layer
volumeBindingMode: WaitForFirstConsumer
parameters:
  guaranteedReadWriteLatency: "true" # provider-specific
  fsType: ext4
```


To use the above Storage Class, define the below Persistent Volume Claim:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow
  selector:
    matchLabels:
      release: "stable"
    matchExpressions:
      - {key: environment, operator: In, values: [dev]}
```