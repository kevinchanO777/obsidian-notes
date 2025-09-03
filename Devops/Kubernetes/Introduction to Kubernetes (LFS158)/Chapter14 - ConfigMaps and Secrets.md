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


[ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/) allow us to decouple the configuration details from the container image. Using ConfigMaps, we pass configuration data as key-value pairs, which are consumed by Pods or any other system components and controllers, in the form of environment variables, sets of commands and arguments, or volumes. We can create ConfigMaps from literal values, from configuration files, from one or more files or directories.


## Create ConfigMap

###  Create ConfigMap - Literal Values

A ConfigMap can be created with the imperative **kubectl create configmap** command, and we can display its details using the **kubectl get** or **kubectl describe** commands.


```bash
# Create configmap
kubectl create configmap my-config \
  --from-literal=key1=value1 \
  --from-literal=key2=value2
  
# View configmap
kubectl get cm my-config -o yaml
```

*output*
```yaml
apiVersion: v1
data:
  key1: value1
  key2: value2
kind: ConfigMap
metadata:
  creationTimestamp: 2024-03-02T07:21:55Z
  name: my-config
  namespace: default
  resourceVersion: "241345"
  selfLink: /api/v1/namespaces/default/configmaps/my-config
  uid: d35f0a3d-45d1-11e7-9e62-080027a46057
```


###  Create ConfigMap - Manifest

`customer1-configmap.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: customer1
data:
  TEXT1: Customer1_Company
  TEXT2: Welcomes You
  COMPANY: Customer1 Company Technology Pct. Ltd.
```


###  Create ConfigMap - From File

Let say we have a file named `permission-reset.properties` 

`permission-reset.properties`
```
permission=read-only
allowed="true"
resetCount=3
```


Then we can create a configmap using:
```bash
kubectl create configmap permission-config \
  --from-file=<path/to/>permission-reset.properties
```


## Use ConfigMap

### Inside Pods: As Environment Variables

In the following example the **myapp-specific-container** Container's environment variables receive their values from specific key-value pairs from two separate ConfigMaps, **config-map-1** and **config-map-2** respectively:

```yaml
...
  containers:
  - name: myapp-specific-container
    image: myapp
    env:
    - name: SPECIFIC_ENV_VAR1
      valueFrom:
        configMapKeyRef:
          name: config-map-1
          key: SPECIFIC_DATA
    - name: SPECIFIC_ENV_VAR2
      valueFrom:
        configMapKeyRef:
          name: config-map-2
          key: SPECIFIC_INFO
...
```