# Pod 

## Displaying only pod names

```
kubectl get pods -o name
pod/nginx
pod/nginx-676b6c5bbc-jglkt
```

## Pod with 02 containers


Create a multi-container pod with 2 container


```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: yellow
  name: yellow
spec:
  containers:
  - image: busybox
    name: lemon
    command: ["sleep","1000"]
  - image: redis
    name: gold
```


```
controlplane ~ ➜  kubectl apply -f pod.yaml 
pod/yellow created


controlplane ~ ➜  kubectl get pods -w
NAME        READY   STATUS    RESTARTS   AGE
app         1/1     Running   0          12m
blue        2/2     Running   0          11m
fluent-ui   1/1     Running   0          12m
red         3/3     Running   0          12m
yellow      2/2     Running   0          6s
```

## Checking pod's log

```
➜  kubectl -n elastic-stack logs kibana
```


## kubectl exec

```
➜  kubectl exec app -n elastic-stack -- cat /log/app.log
```

## Another pod with 2 containers

This pod has 2 containers and persistent volumes mounted.

```
---
apiVersion: v1
kind: Pod
metadata:
  name: app
  namespace: elastic-stack
  labels:
    name: app
spec:
  containers:
  - name: app
    image: kodekloud/event-simulator
    volumeMounts:
    - mountPath: /log
      name: log-volume

  - name: sidecar
    image: kodekloud/filebeat-configured
    volumeMounts:
    - mountPath: /var/log/event-simulator/
      name: log-volume

  volumes:
  - name: log-volume
    hostPath:
      # directory location on host
      path: /var/log/webapp
      # this field is optional
      type: DirectoryOrCreate
```



## Init Containers

In a multi-container pod, each container is expected to run a process that stays alive as long as the POD's lifecycle. For example in the multi-container pod that we talked about earlier that has a web application and logging agent, both the containers are expected to stay alive at all times. The process running in the log agent container is expected to stay alive as long as the web application is running. If any of them fails, the POD restarts.



But at times you may want to run a process that runs to completion in a container. For example a process that pulls a code or binary from a repository that will be used by the main web application. That is a task that will be run only one time when the pod is first created. Or a process that waits for an external service or database to be up before the actual application starts. That's where initContainers comes in.



An initContainer is configured in a pod like all other containers, except that it is specified inside a initContainers section, like this:


```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ;']
```

