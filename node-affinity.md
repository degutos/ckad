# Node Affinity


## What is Node Affinity 


`Node Affinity` in Kubernetes is a more flexible and advanced way to schedule pods on specific nodes based on node labels. It allows you to specify rules for how pods should be placed on nodes, and it provides more complex logic compared to Node Selector.

`Node Affinity` is part of the affinity feature in Kubernetes, which provides rules for controlling where a pod can run based on conditions like node labels. It is often used when you want to enforce more complex scheduling policies beyond the simple key-value matching that Node Selector provides.

## Key Concepts of Node Affinity:

### Node Affinity vs. Node Selector:

`Node Selector` is simpler and only allows key-value matching.

`Node Affinity` is more flexible and allows complex matching rules, such as using operators (In, NotIn, Exists, DoesNotExist) and combining multiple conditions.
Node Affinity Types:

`RequiredDuringSchedulingIgnoredDuringExecution`: This is the hard constraint. The pod will only be scheduled onto nodes that match the affinity rules.

`PreferredDuringSchedulingIgnoredDuringExecution`: This is the soft constraint. Kubernetes will try to schedule the pod on a node that matches the affinity rules, but if no such nodes are available, the pod may be scheduled elsewhere.

### Key Components of Node Affinity:

`key`: The node label key you want to match.
`operator`: The operator that defines the matching logic. It can be one of:
`In`: The key's value must be in the specified list of values.
`NotIn`: The key's value must not be in the specified list of values.
`Exists`: The key must be present on the node (value is not important).
`DoesNotExist`: The key must not be present on the node.
`values`: The list of values for the key that the node must match when using In or NotIn.


### Example of Node Affinity in Pod YAML:

Here’s an example of how you can use Node Affinity to control pod placement based on node labels.

Example: Required Node Affinity

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
          - key: zone
            operator: In
            values:
            - us-west
  containers:
  - name: my-container
    image: nginx
```


In this example:

The `preferredDuringSchedulingIgnoredDuringExecution` means that Kubernetes will prefer scheduling the pod onto nodes with `disktype=ssd` and `zone=us-west`, but it is not a hard requirement.

The weight indicates the preference's importance. Kubernetes will try to satisfy the node affinity rules with higher weights first.

### Example: Preferred Node Affinity

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
      - weight: 2
        preference:
          matchExpressions:
          - key: zone
            operator: In
            values:
            - us-west
  containers:
  - name: my-container
    image: nginx
```

In this example:

The `preferredDuringSchedulingIgnoredDuringExecution` means that Kubernetes will prefer scheduling the pod onto nodes with `disktype=ssd` and `zone=us-west`, but it is not a hard requirement.

The weight indicates the preference's importance. Kubernetes will try to satisfy the node affinity rules with higher weights first.


### Checking labels on node 

```
✖ kubectl describe no node01
Name:               node01
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=node01
                    kubernetes.io/os=linux
...
```

Another way to show labels

```
➜  kubectl get no node01 --show-labels
NAME     STATUS   ROLES    AGE   VERSION   LABELS
node01   Ready    <none>   13m   v1.31.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node01,kubernetes.io/os=linux
```

And one more way 

```
➜  kubectl get no node01 -o custom-columns=LABELS:metadata.labels
LABELS
map[beta.kubernetes.io/arch:amd64 beta.kubernetes.io/os:linux kubernetes.io/arch:amd64 kubernetes.io/hostname:node01 kubernetes.io/os:linux]
```




### Adding a new label to a node.

```
kubectl label node node01 color=blue
```

### Create a deployment with 3 replicas

```
kubectl create deployment blue --image=nginx --replicas=3
```

Notice where those new pods are running on node01 and controlplane:

```
➜  kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
blue-6dc9b889f5-8gtsr   1/1     Running   0          34s   172.17.1.2   node01         <none>           <none>
blue-6dc9b889f5-nb4v6   1/1     Running   0          34s   172.17.1.3   node01         <none>           <none>
blue-6dc9b889f5-xrtjb   1/1     Running   0          34s   172.17.0.4   controlplane   <none>           <none>
```

### Create a Node affinity to deployment to place pods on node01

```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: blue
  name: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: blue
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: blue
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: color
                operator: In
                values:
                - blue
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

Notice that all pods are running now on node01

```
➜  kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
blue-5659879b55-4fg92   1/1     Running   0          17s   172.17.1.5   node01   <none>           <none>
blue-5659879b55-t24fc   1/1     Running   0          15s   172.17.1.6   node01   <none>           <none>
blue-5659879b55-vcsqs   1/1     Running   0          18s   172.17.1.4   node01   <none>           <none>

```

### Create deployment to place pods on controlplane

Create a new deployment named red with the nginx image and 2 replicas, and ensure it gets placed on the controlplane node only.

Use the label key - node-role.kubernetes.io/control-plane - which is already set on the controlplane node.


```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: red
  name: red
spec:
  replicas: 2
  selector:
    matchLabels:
      app: red
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: red
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/control-plane
                operator: Exists
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

```
controlplane ~ ➜  kubectl apply -f deploy1.yaml 
deployment.apps/red created
```

Notice that the new pods red place on controlplane

```
controlplane ~ ➜  kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
blue-5659879b55-4fg92   1/1     Running   0          6m17s   172.17.1.5   node01         <none>           <none>
blue-5659879b55-t24fc   1/1     Running   0          6m15s   172.17.1.6   node01         <none>           <none>
blue-5659879b55-vcsqs   1/1     Running   0          6m18s   172.17.1.4   node01         <none>           <none>
red-7c64d97ff5-2t54g    1/1     Running   0          7s      172.17.0.5   controlplane   <none>           <none>
red-7c64d97ff5-vrz5q    1/1     Running   0          7s      172.17.0.6   controlplane   <none>           <none>
```
