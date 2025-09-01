

To access the application, a user or another application needs to connect to a Pod running the target application. As Pods are ephemeral in nature, resources like IP addresses allocated to them cannot be static. Pods could be terminated abruptly or be rescheduled based on existing requirements.

-> IP address of new pod changes
![[Pasted image 20250901182133.png]]

To overcome this situation, Kubernetes provides a higher-level abstraction called _Service_, which logically groups Pods and defines a policy to access them. This grouping is achieved via _Labels_ and _Selectors_. This logical grouping strategy is used by Pod controllers, such as ReplicaSets, Deployments, and even DaemonSets. Below is a Deployment definition manifest for the **frontend** app, to aid with the correlation of Labels, Selectors, and port values between the Deployment controller, its Pod replicas, and the Service definition manifest presented in an upcoming section.

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
        - image: frontend-application
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

### Service Object Example

The following is an example of a Service object definition. This represents the declarative method to define an object, and can serve as a template for a much more complex Service definition manifest if desired. By omitting the Service **type** from the definition manifest, we create the default service type, the **ClusterIP** type (the ClusterIP Service type will be covered in an upcoming lesson).

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