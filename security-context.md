# Security Context


### How to find out the user-id within the container

```
controlplane ~ ➜  kubectl get pods 
NAME             READY   STATUS    RESTARTS   AGE
ubuntu-sleeper   1/1     Running   0          41s

controlplane ~ ➜  kubectl exec ubuntu-sleeper -- whoami
root
```
>Note: as we can see the user root is the owner of the container or process running within the container


### Enabling the pod and the process to run as user 1010

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2023-02-12T17:37:00Z"
  name: ubuntu-sleeper
  namespace: default
  resourceVersion: "760"
  uid: 796beac3-e465-461c-9657-be61fe3e1b5d
spec:
  securityContext:
    runAsUser: 1010
  containers:
  - command:
    - sleep
    - "4800"
```

```
controlplane ~ ➜  k exec -it ubuntu-sleeper -- sh
$ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
1010           1  0.0  0.0   2784  1096 ?        Ss   17:48   0:00 sleep 4800
1010          87  0.0  0.0   2884  1004 pts/0    Ss   17:51   0:00 sh
1010          93  0.0  0.0   7056  1596 pts/0    R+   17:51   0:00 ps aux

We can see the process sleep is running as user 1010
```

We can also see with this command:

```
controlplane ~ ✖ kubectl exec ubuntu-sleeper -- ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
1010           1  0.0  0.0   2696  1072 ?        Ss   09:44   0:00 sleep 4800
1010          68  0.0  0.0   7888  4052 ?        Rs   09:47   0:00 ps aux
```


Also same user 1010 on Host node

```
controlplane ~ ➜  ps aux | grep sleep
11575 1010      0:00 sleep 4800
```

Lets see another example:

```
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  securityContext:
    runAsUser: 1001
  containers:
  -  image: ubuntu
     name: web
     command: ["sleep", "5000"]
     securityContext:
      runAsUser: 1002

  -  image: ubuntu
     name: sidecar
     command: ["sleep", "5000"]

```

In this case above we see the container rule of user 1002 will take place and not the pod rule for user 1001. The sidecar container will have process running as user 1001.

Basically the above yaml has two securityContext configured, one at the pod level and another one at the container level, the container level will always overright the pod level. In this case the container web will run as user 1002 and the container sidecar has not securityContext configured, so it will inherit from the securityContext at the pod level.



### How to add securityContext capabilities 

```
controlplane ~ ➜  kubectl get pods ubuntu-sleeper -o yaml | grep securityContext: -A3 -B08
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    imagePullPolicy: Always
    name: ubuntu
    resources: {}
    securityContext:
      capabilities:
        add:
        - SYS_TIME
```

Note that we need to add the capabilities under the container level.

This is our sample of our yaml file:

```
controlplane ~ ➜  cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2025-01-31T09:38:06Z"
  name: ubuntu-sleeper
  namespace: default
  resourceVersion: "735"
  uid: e71e92e7-ff71-4169-9fae-06e095def722
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    imagePullPolicy: Always
    name: ubuntu
    securityContext:
      capabilities:
        add: ["SYS_TIME"]
...
```

To add two capabilities use this format:

```
    name: ubuntu
    securityContext:
      capabilities:
        add: ["SYS_TIME", "NET_ADMIN"]
```






