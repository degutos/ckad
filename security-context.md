# Security Context

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

In this case above we see the container rule of user 1002 will take place and not the pod rule for user 1001. The sidecar container will have process running as user 1001 





