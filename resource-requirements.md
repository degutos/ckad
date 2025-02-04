# Resource Requirements


  

- Pod kube-scheduler chooses where each pod will be assigned to, the best node to schedule the new pod.
- if node has no resource available the scheduler will assign the pod to a new node
- if all nodes are full you will see the new pod failing with error `Insufficient cpu` 
- We can have resources request for cpu and mem


## Resource request

Resource Requests are the minimum amount of CPU and memory that a container in a pod is guaranteed to have. They are part of the resource management system in Kubernetes, which helps ensure that containers get the resources they need to run efficiently without overwhelming the node or cluster.

- CPU
- Mem
- we can specify a resource request of CPU and Memory for each pod
- We can specify a minimum of resource of for example cpu=1 and mem=1 
- When scheduler try to schedule a pod to a node it takes in consideration the amount of resources request to assign a node to those pods.
- CPU Requests: This is the minimum amount of CPU that the container is guaranteed to receive. It's specified in CPU units (like 500m for 0.5 CPUs or 1 for 1 full CPU - 1 cpu is 1 AWS vCPU or 1 GCP CORE or 1 azure core or 1 hyperthread ), the minimum amount you can assign to a container is 0.1 or 100m. Kubernetes will ensure that the container gets at least the amount of CPU defined in the request.

Memory Requests: This is the minimum amount of memory that the container is guaranteed to have. It’s usually specified in bytes (like 64Mi for 64 megabytes or 1Gi for 1 gigabyte). Kubernetes ensures that the container has at least this much memory available to it.
- We can specify in our yaml file a amount of resources requests:

```
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "500m"

```

## Resource limit.

In Kubernetes, a resource limit is the maximum amount of CPU or memory that a container in a pod is allowed to use. Unlike resource requests, which define the minimum amount of resources the container will get, resource limits define the upper boundary. If the container tries to use more resources than the defined limit, Kubernetes will impose restrictions, such as throttling or terminating the container, to prevent it from using more than the allocated resources.

Key Points:

CPU Limit: The maximum amount of CPU the container is allowed to use. If the container tries to use more CPU than its limit, it will be throttled. CPU limits are set in units of CPU cores (e.g., 500m for 0.5 CPUs, 1 for 1 full CPU).

Memory Limit: The maximum amount of memory the container can use. If the container exceeds this limit, it will be killed (terminated with an error `"OOM (OUT OF MEMORY)"`) and possibly restarted by Kubernetes. Memory limits are typically set in megabytes (e.g., 256Mi for 256 MB, 1Gi for 1 GB).

Example:
Here’s an example of a Kubernetes pod specification that includes both requests and limits for CPU and memory:

```
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "500m"
      limits:
        memory: "128Mi"
        cpu: "1"

```

Note:

- By default kubernetes has not resource requirements or resource limit set to any container which means each pod can consumes as much resources they need in any node and this can cause issues to other pods. 

## Limit Range

Explanation:

- type: Container: This means the limit range applies to the containers within the namespace.

- max.cpu: Sets the maximum CPU limit for any container in the specified namespace (in this case, up to 2 CPUs).

- min.cpu: Sets the minimum CPU request for any container in the namespace (in this case, 200m or 0.2 CPUs).
default.cpu: If a container does not specify a CPU request, this will be the default request value for that container (500m or 0.5 CPUs).

- defaultRequest.cpu: This is the default CPU request for containers that do not explicitly define one. If not specified, Kubernetes uses this value to ensure a reasonable allocation.


```
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
  namespace: default  # Specify your desired namespace here
spec:
  limits:
  - type: Container
    max:
      cpu: "2"  # Maximum CPU limit for containers (2 CPUs)
    min:
      cpu: "200m"  # Minimum CPU request for containers (200m or 0.2 CPUs)
    default:
      cpu: "500m"  # Default CPU request (if none is specified)
    defaultRequest:
      cpu: "500m"  # Default CPU request for containers
    resource: cpu

```

Usage:
To apply this LimitRange, you would run:

```
kubectl apply -f limit-range-cpu.yaml

```

Note:

- We can also set quota for each namespace. 



## Sample of Requirements and limits

```controlplane ~ ➜  kubectl describe po rabbit
Name:             rabbit
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.243.140
Start Time:       Tue, 04 Feb 2025 06:52:16 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.42.0.9
IPs:
  IP:  10.42.0.9
Containers:
  cpu-stress:
    Container ID:  containerd://93c1bb30e8d45d2b4b393c7fc58cf8be9c3fed6a7e6525c206ce8c4397bd06af
    Image:         ubuntu
    Image ID:      docker.io/library/ubuntu@sha256:72297848456d5d37d1262630108ab308d3e9ec7ed1c3286a32fe09856619a782
    Port:          <none>
    Host Port:     <none>
    Args:
      sleep
      1000
    State:          Running
      Started:      Tue, 04 Feb 2025 06:52:19 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:  1
    Requests:
      cpu:        500m
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-5gwg7 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-5gwg7:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  43s   default-scheduler  Successfully assigned default/rabbit to controlplane
  Normal  Pulling    42s   kubelet            Pulling image "ubuntu"
  Normal  Pulled     41s   kubelet            Successfully pulled image "ubuntu" in 1.413s (1.413s including waiting). Image size: 29763699 bytes.
  Normal  Created    41s   kubelet            Created container cpu-stress
  Normal  Started    40s   kubelet            Started container cpu-stress
```

- Notice that the CPU requirements set on this above Pod is 500m or 0.5 


### OOM

```
controlplane ~ ➜  kubectl get pods
NAME       READY   STATUS      RESTARTS      AGE
elephant   0/1     OOMKilled   2 (19s ago)   21s

```


- Lets describe this pod 

```
controlplane ~ ➜  kubectl describe po elephant 
Name:             elephant
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.243.140
Start Time:       Tue, 04 Feb 2025 06:58:10 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.42.0.10
IPs:
  IP:  10.42.0.10
Containers:
  mem-stress:
    Container ID:  containerd://bc72c1e206df06c943d0d4dccb59b589b2a5f5f19f338a2c84b5fe1a06860db9
    Image:         polinux/stress
    Image ID:      docker.io/polinux/stress@sha256:b6144f84f9c15dac80deb48d3a646b55c7043ab1d83ea0a697c09097aaad21aa
    Port:          <none>
    Host Port:     <none>
    Command:
      stress
    Args:
      --vm
      1
      --vm-bytes
      15M
      --vm-hang
      1
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       OOMKilled
      Exit Code:    1
      Started:      Tue, 04 Feb 2025 06:58:53 +0000
      Finished:     Tue, 04 Feb 2025 06:58:53 +0000
    Ready:          False
    Restart Count:  3
    Limits:
      memory:  10Mi
    Requests:
      memory:     5Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-g8947 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       False 
  ContainersReady             False 
  PodScheduled                True 
Volumes:
  kube-api-access-g8947:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  70s                default-scheduler  Successfully assigned default/elephant to controlplane
  Normal   Pulled     69s                kubelet            Successfully pulled image "polinux/stress" in 647ms (647ms including waiting). Image size: 4041495 bytes.
  Normal   Pulled     52s                kubelet            Successfully pulled image "polinux/stress" in 132ms (132ms including waiting). Image size: 4041495 bytes.
  Normal   Pulling    27s (x4 over 70s)  kubelet            Pulling image "polinux/stress"
  Normal   Created    27s (x4 over 69s)  kubelet            Created container mem-stress
  Normal   Started    27s (x4 over 69s)  kubelet            Started container mem-stress
  Normal   Pulled     27s (x2 over 68s)  kubelet            Successfully pulled image "polinux/stress" in 138ms (138ms including waiting). Image size: 4041495 bytes.
  Warning  BackOff    11s (x6 over 67s)  kubelet            Back-off restarting failed container mem-stress in pod elephant_default(2a4ebaaf-1662-413b-a491-60ec833ec8a7)
```

- As we see the status of the pod is Reason:       `OOMKilled`. The status OOMKilled indicates that it is failing because the pod ran out of memory.
  

- The elephant pod runs a process that consumes 15Mi of memory. Lets Increase the limit of the elephant pod to 20Mi.


```
kubectl get po elephant -o yaml > pod.yaml
```

```
vi pod.yaml 
```

- Make a change to:

```
    resources:
      limits:
        memory: 20Mi
```

- Lets delete the pod and apply the new configuration

```
kubectl delete po elephant
```

```
kubectl apply -f pod.yaml 
pod/elephant created

 kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
elephant   1/1     Running   0          7s

```










