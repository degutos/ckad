# Node Selector

Kubernetes can always schedule a pod anywhere in the cluster since the node has sufficient capacity. Sometimes you want a pod with high workload to be placed on a large hardware (node on cluster), in this case we can use node selector.

In Kubernetes, a Node Selector is a way to constrain which nodes a pod can be scheduled on based on node labels. Essentially, it allows you to select nodes with specific characteristics or properties by filtering them using labels.

## Key Concepts:

`Node Labels`: Labels are key-value pairs attached to nodes that describe the properties or attributes of a node (e.g., zone=us-west, gpu=true, tier=backend). These labels help Kubernetes distinguish between different nodes in the cluster.

`Node Selector`: It’s a field within the pod specification that specifies which node labels a node must have for the pod to be scheduled onto it.

## How Node Selector Works:

When you define a Node Selector in a pod's specification, Kubernetes will only schedule the pod on nodes that match the given labels.

This provides a simple way to constrain pods to certain groups of nodes without having to use complex scheduling policies.

## Syntax Example:
Here’s an example of a Kubernetes pod YAML file that uses a Node Selector to schedule the pod on nodes with a specific label.

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  nodeSelector:
    disktype: ssd  # Node selector for nodes with label "disktype=ssd"
  containers:
  - name: my-container
    image: nginx
```

Explanation:

`nodeSelector:` In this case, the pod will be scheduled only on nodes that have the label disktype=ssd.

`disktype: ssd:` This is a label that the nodes must have to allow the pod to be scheduled on them.


## Node Selector vs. Taints & Tolerations:


`Node Selector` is used for directly selecting nodes based on labels.

`Taints and Tolerations` are more of a repellent mechanism that allows you to avoid scheduling certain pods on nodes or, conversely, allow certain pods to run on tainted nodes.

`Node Selector` is more about `"where"` you want the pod to go, while `Taints & Tolerations` are more about avoiding certain nodes unless a pod is specifically marked to tolerate those conditions.

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  nodeSelector:
    disktype: ssd  # The node must have the "disktype=ssd" label
    zone: us-west  # The node must also have the "zone=us-west" label
  containers:
  - name: my-container
    image: nginx

```

In this case:

The pod will only be scheduled on nodes that have both the labels disktype=ssd and zone=us-west.


##  Node selector limitation:

- Node selector has some limitation for example:
  - LARGE OR SMALL - you can not use use node selector for two labels
  - NOT SMALL - you can not say a node selector with negative.
  - In this case we can use another resource in kubernetes called `Node Affinity`



