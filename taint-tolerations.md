# Taint and tolerations

- Taint are applied to nodes
- Tolerations are applied to pods
  

In Kubernetes, a taint is a mechanism used to mark a node so that no pods can be scheduled onto it unless the pod has a toleration that matches the taint. Taints allow you to control where your pods are allowed to run by adding constraints on the nodes. This is useful for managing nodes that have special purposes, conditions, or resources that certain pods should avoid or specifically target.

## Key Concepts:

Taint: A label applied to a node that repels pods from being scheduled onto that node unless the pod has a matching toleration. Taints are applied to nodes to "taint" them in some way.

Toleration: A property applied to a pod that allows it to be scheduled on a node with a matching taint. A toleration is a way to allow a pod to "tolerate" a taint, meaning the pod is allowed to run on the tainted node.

## Taint Format:

A taint consists of three parts:

`Key`: A label key (like "key=value").

`Value`: The label value that pairs with the key (like "value").

`Effect`: The impact of the taint on the scheduling of the pod. It can be one of the following:

`NoSchedule`: The pod will not be scheduled on the node unless it has a matching toleration.

`PreferNoSchedule`: Kubernetes will try to avoid scheduling the pod on the node, but it’s not a hard requirement. The scheduler may place the pod there if no other nodes are available.

`NoExecute`: Pods that are already running on the node will be evicted if they don’t have a matching toleration. New pods will not be scheduled on the node.


```
kubectl taint nodes node-name key=value:NoSchedule
```

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
  containers:
  - name: my-container
    image: nginx
```

## Check if node is tainted or not

```
controlplane ~ ➜  kubectl describe no node01
Name:               node01
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=node01
                    kubernetes.io/os=linux
Annotations:        flannel.alpha.coreos.com/backend-data: {"VNI":1,"VtepMAC":"c2:7f:62:14:0b:c5"}
                    flannel.alpha.coreos.com/backend-type: vxlan
                    flannel.alpha.coreos.com/kube-subnet-manager: true
                    flannel.alpha.coreos.com/public-ip: 192.168.212.159
                    kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/containerd/containerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Wed, 05 Feb 2025 05:25:44 +0000
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  node01
  AcquireTime:     <unset>
  RenewTime:       Wed, 05 Feb 2025 05:32:01 +0000
```


- Checking if controlplane has any taints

```
➜  kubectl describe no controlplane | grep -i taint
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
```



## Taint a node


Create a taint on node01 with key of spray, value of mortein and effect of NoSchedule


```
kubectl taint node node01 spray=mortein:NoSchedule
node/node01 tainted
```

```
kubectl describe no node01
Name:               node01
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=node01
                    kubernetes.io/os=linux
Annotations:        flannel.alpha.coreos.com/backend-data: {"VNI":1,"VtepMAC":"c2:7f:62:14:0b:c5"}
                    flannel.alpha.coreos.com/backend-type: vxlan
                    flannel.alpha.coreos.com/kube-subnet-manager: true
                    flannel.alpha.coreos.com/public-ip: 192.168.212.159
                    kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/containerd/containerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Wed, 05 Feb 2025 05:25:44 +0000
Taints:             spray=mortein:NoSchedule
Unschedulable:      false
```

- Lets create a pod 

```
 kubectl run mosquito --image=nginx
pod/mosquito created

kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
mosquito   0/1     Pending   0          18s

```

- As we see the pod mosquito is in `Pending` state.
  
```
kubectl describe po mosquito 
Name:             mosquito
Namespace:        default
Priority:         0
Service Account:  default
Node:             <none>
Labels:           run=mosquito
Annotations:      <none>
Status:           Pending
...
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  40s   default-scheduler  0/2 nodes are available: 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }, 1 node(s) had untolerated taint {spray: mortein}. preemption: 0/2 nodes are available: 2 Preemption is not helpful for scheduling.
```

- The pod created `mosquito` can not tolerate the taint spray=mortein 


## Create a pod with toleration

Create another pod named bee with the nginx image, which has a toleration set to the taint mortein.

```
kubectl run bee --image=nginx --dry-run=client -o yaml > bee.yaml

$ cat bee.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: bee
  name: bee
spec:
  containers:
  - image: nginx
    name: bee
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Lets add toleration to this pod yaml file

```
➜  cat bee.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: bee
  name: bee
spec:
  tolerations:
    - key: "spray"
      operator: "Equal"
      value: "mortein"
      effect: "NoSchedule"
  containers:
  - image: nginx
    name: bee
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

- Lets apply new bee.yaml 

```
➜  kubectl apply -f bee.yaml 
pod/bee created

➜  kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
bee        1/1     Running   0          9s
mosquito   0/1     Pending   0          12m
```

- Notice that the new pod bee landed into a node01 even with the taint

```
➜  kubectl get pod -o wide
NAME       READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
bee        1/1     Running   0          97s   172.17.1.2   node01   <none>           <none>
mosquito   0/1     Pending   0          13m   <none>       <none>   <none>           <none>
```


## Removing a taint from a node

```
➜  kubectl describe no controlplane | grep -i taint
Taints:             node-role.kubernetes.io/control-plane:NoSchedule


kubectl taint node controlplane node-role.kubernetes.io/control-plane:NoSchedule-
node/controlplane untainted
```

- Notice that we used the same taint name and added a `-` at the end to remove the taint. 

- After removing the taint from the controlplane, this node will be available for pods

```
➜  kubectl get pods -o wide
NAME       READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
bee        1/1     Running   0          10m   172.17.1.2   node01         <none>           <none>
mosquito   1/1     Running   0          22m   172.17.0.4   controlplane   <none>           <none>
```

- Notice that the pod mosquito landed on `controlplane` node.

