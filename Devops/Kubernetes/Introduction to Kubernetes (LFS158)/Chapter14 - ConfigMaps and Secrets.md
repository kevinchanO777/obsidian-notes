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


### Inside Pods: As Volumes

We can mount a **vol-config-map** ConfigMap as a Volume inside a Pod. The **configMap** Volume plugin converts the ConfigMap object into a mountable resource. For each key in the ConfigMap, a file gets created in the mount path (where the file is named with the key name) and the respective key's value becomes the content of the file:

```yaml
...
  containers:
  - name: myapp-vol-container
    image: myapp
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: vol-config-map
...
```

For more details, please explore the documentation on using [ConfigMaps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/).


## Using ConfigMaps as Volumes Demo

This exercise guide was prepared for the video demonstration available in this chapter. It includes an **index.html** file and a Deployment definition manifest that can be used as templates to define other similar objects as needed. The goal of the demo is to store the custom webserver **index.html** file in a ConfigMap object, which is mounted by the nginx container specified by the Pod template nested in the Deployment definition manifest.

`index.html`
```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to GREEN App!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
        background-color: GREEN;
    }
</style>
</head>
<body>
<h1 style=\"text-align: center;\">Welcome to GREEN App!</h1>
</body>
</html>
```

`web-green-with-cm.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: green-web
  name: green-web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: green-web
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: green-web
    spec:
      volumes:
      - name: web-config
        configMap:
          name: green-web-cm
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: web-config
status: {}
```

1. Create configmap with `kubectl create cm green-web-cm --from-file=index.html`
2. The deployment manifest defined the volume type configmap and mount that to the pod. We only need to run `kubectl apply -f web-green-with-cm.yaml`

Forward a local port to the port 80 of the green app pod and you should see a green webpage. 

Or

`kubectl expose deploy green-web --name green-web-svc --type NodePort` as a service


## Secret
