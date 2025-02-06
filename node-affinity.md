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