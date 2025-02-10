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
