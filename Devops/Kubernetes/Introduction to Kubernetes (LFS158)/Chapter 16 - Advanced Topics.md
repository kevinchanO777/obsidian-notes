
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
# Annotations

[Annotations](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/) allow us to attach arbitrary non-identifying metadata to any objects, in a key-value format:

```json
"annotations": {
  "key1": "value1",
  "key2": "value2"
}
```

We can easily annotate an existing object, a Pod for example, as such:

```bash
kubectl annotate pod mypod key1=value1 key2=value2
```

Unlike Labels, annotations are not used to identify and select objects. Annotations can be used to:

- Store build/release IDs, PR numbers, git branch, etc.
- Phone/pager numbers of people responsible, or directory entries specifying where such information can be found.
- Pointers to logging, monitoring, analytics, audit repositories, debugging tools, etc.
- Ingress controller information.
- Deployment state and revision information.

For example, while creating a Deployment, we can add a description as seen below:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webserver
  annotations:
    description: Deployment based PoC dates 2nd Mar'2022
....
```

Imperatively, an object can be annotated with its latest configuration using the **--save-config=true** option. Try the two commands presented below to test the feature’s effect, then try the same two commands again without the **--save-config** option to see the difference:

```bash
kubectl run saved --image=nginx:alpine --save-config=true

kubectl get pod saved -o yaml
```


# Quota and Limits Management

When there are many users sharing a given Kubernetes cluster, there is always a concern for fair usage. A user should not take undue advantage. To address this concern, administrators can use the [ResourceQuota](https://kubernetes.io/docs/concepts/policy/resource-quotas/) API resource, which provides constraints that limit aggregate resource consumption per Namespace.

We can set the following types of quotas per Namespace:

- **Compute Resource Quota  
    **We can limit the total sum of compute resources (CPU, memory, etc.) that can be requested in a given Namespace.
- **Storage Resource Quota  
    **We can limit the total sum of storage resources (PersistentVolumeClaims, requests.storage, etc.) that can be requested.
- **Object Count Quota  
    **We can restrict the number of objects of a given type (Pods, ConfigMaps, PersistentVolumeClaims, ReplicationControllers, Services, Secrets, etc.).

Examine the two ResourceQuota manifests below. They exemplifying compute resources quotas (requests and limits) in a given namespace **myspace**:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
  namespace: myspace
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    requests.nvidia.com/gpu: 4:<mark style="background: #ADCCFFA6;"></mark>
```

and various Kubernetes object counts quotas (ConfigMaps, PVCs, Pods, Services in general, Load Balancer Service types, etc.) in a given namespace **myspace**:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: object-counts
  namespace: myspace
spec:
  hard:
    configmaps: "10"
    persistentvolumeclaims: "4"
    pods: "4"
    secrets: "10"
    services: "10"
    services.loadbalancers: "2"
```

An additional resource that helps limit resource allocation to pods and containers in a namespace, is the [LimitRange](https://kubernetes.io/docs/concepts/policy/limit-range/), used in conjunction with the ResourceQuota API resource. A LimitRange can:

- Set compute resources usage limits per Pod or Container in a namespace.
- Set storage request limits per PersistentVolumeClaim in a namespace.
- Set a request to limit ratio for a resource in a namespace.
- Set default requests and limits and automatically inject them into Containers' environments at runtime.

Examine the LimitRange manifest below. It exemplifies how CPU constraints are enforced for individual Containers of Pods running in the **myspace** namespace:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-constraint-per-container
  namespace: myspace
spec:
  limits:
  - default:            # default limits
      cpu: 500m
    defaultRequest:     # default requests
      cpu: 500m
    max:                # max defines the highest value of the range
      cpu: "1"
    min:                # min defines the lowest value of the range
      cpu: 100m
    type: Container
```