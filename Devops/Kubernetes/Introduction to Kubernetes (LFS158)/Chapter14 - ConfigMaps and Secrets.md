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

Let's assume that we have a _Wordpress_ blog application, in which our **wordpress** frontend connects to the **MySQL** database backend using a password. While creating the Deployment for **wordpress**, we can include the **MySQL** password in the Deployment's YAML definition manifest, but the password would not be protected. The password would be available to anyone who has access to the definition manifest.

In this scenario, the [Secret](https://kubernetes.io/docs/concepts/configuration/secret/) object can help by allowing us to encode in base 64 the sensitive information before sharing it. We can share sensitive information like passwords, tokens, or keys in the form of key-value pairs, similar to ConfigMaps; thus, we can control how the information in a Secret is used, reducing the risk for accidental exposures. In Deployments or other resources, the Secret object is _referenced_, without exposing its content.

It is important to keep in mind that by default, the Secret data is stored as plain text inside **etcd**, therefore administrators must limit access to the API server and **etcd**. <mark style="background: #FF5582A6;">However, Secret data can be encrypted at rest while it is stored in **etcd**, but this feature needs to be enabled at the API server level by the Kubernetes cluster administrator.</mark>


### Create a Secret from Literal Values

To create a Secret, we can use the imperative **kubectl create secret** command:

```bash
kubectl create sec generic my-password --from-literal=password=mysqlpassword
```

The above command would create a secret called **my-password**, which has the value of the **password** key set to **mysqlpassword**. `Opaque` is the default Secret type if you don't explicitly specify a type in a Secret manifest. When you create a Secret using `kubectl`, you must use the `generic` subcommand to indicate an `Opaque` Secret type

After successfully creating a secret we can analyze it with the **get** and **describe** commands. They do not reveal the content of the Secret. The type is listed as **Opaque**.

```bash
kubectl get sec my-password
```

**NAME          TYPE     DATA   AGE**  
**my-password   Opaque   1      8m**


```bash
kubectl describe sec my-password
```


### Create a Secret from a Definition Manifest

We can create a Secret manually from a YAML definition manifest. The example manifest below is named **mypass.yaml**. There are two types of maps for sensitive information inside a Secret: 

1. data - base64 encoded data
2. stringData - string

With <mark style="background: #ADCCFFA6;">data</mark> maps, each value of a sensitive information field must be encoded using **base64**. If we want to have a definition manifest for our Secret, we must first create the **base64** encoding of our password:

```bash
# Which print out bXlzcWxwYXNzd29yZAo=
echo mysqlpassword | base64
```

and then use it in the definition manifest:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-password
type: Opaque
data:
  password: bXlzcWxwYXNzd29yZAo=
```

Please note that **base64** encoding does not mean encryption, and anyone can easily decode our encoded data. Therefore, make sure you do not commit a Secret's definition file in the source code.

With <mark style="background: #BBFABBA6;">stringData</mark> maps, there is no need to encode the value of each sensitive information field. The value of the sensitive field will be encoded when the **my-password** Secret is created:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-password
type: Opaque
stringData:
  password: mysqlpassword
```

### Create a Secret from a File

First, we prepare a `password.txt` file. Make sure no newline character at the end.
```sh
echo -n 'ohMyGod!!!!!!!' > ./password.txt
```

Now we can create the Secret from the **password.txt** file:
```bash
kubectl create secret generic my-passord2 --from-file=password.txt
```

Default key will be the name of the file. [See](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/#use-source-files)


### Use Secrets Inside Pods: As Environment Variables

Secrets are consumed by Containers in Pods as mounted data volumes, or as environment variables, and are referenced in their entirety (using the **envFrom** heading) or specific key-values (using the **env** heading).

Below we reference only the **password** key of the **my-password** Secret and assign its value to the **WORDPRESS_DB_PASSWORD** environment variable:

```yaml
....
spec:
  containers:
  - image: wordpress:4.7.3-apache
    name: wordpress
    env:
    - name: WORDPRESS_DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: my-password
          key: password
....
```