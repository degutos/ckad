# Ingress 

Ingress Controller, resources and applications

In Kubernetes:

- **Ingress Controller**: It's a component that manages and routes external HTTP(S) traffic to services within the Kubernetes cluster. It acts as a reverse proxy and is responsible for fulfilling the Ingress resources defined in the cluster.

- **Ingress Resource**: It's a Kubernetes API object that defines how external HTTP(S) traffic should be routed to services in the cluster. It specifies rules such as domain names, paths, and which services to route traffic to, allowing you to expose services outside the cluster in a controlled way.

Together, the **Ingress Controller** implements the rules defined in the **Ingress Resource**, enabling efficient management of external access to your services.

## Ingress Controller 

```
➜  kubectl get deploy -n ingress-nginx
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
ingress-nginx-controller   1/1     1            1           4m40s
```

```
 ➜  kubectl get all -A | grep ingress | grep controller
ingress-nginx   pod/ingress-nginx-controller-6c4c749b95-t6c2q   1/1     Running     0          7m1s
ingress-nginx   service/ingress-nginx-controller             NodePort    172.20.249.63    <none>        80:30080/TCP,443:32103/TCP   7m1s
ingress-nginx   service/ingress-nginx-controller-admission   ClusterIP   172.20.113.108   <none>        443/TCP                      7m1s
ingress-nginx   deployment.apps/ingress-nginx-controller   1/1     1            1           7m1s
ingress-nginx   replicaset.apps/ingress-nginx-controller-6c4c749b95   1         1         1       7m1s
```

## Ingress Resource

```
➜  kubectl get ingress -A
NAMESPACE   NAME                 CLASS    HOSTS   ADDRESS         PORTS   AGE
app-space   ingress-wear-watch   <none>   *       172.20.249.63   80      10m
```

Lets describe our ingress resource

```
➜  kubectl describe ingress -n app-space ingress-wear-watch 
Name:             ingress-wear-watch
Labels:           <none>
Namespace:        app-space
Address:          172.20.249.63
Ingress Class:    <none>
Default backend:  <default>
Rules:
  Host        Path  Backends
  ----        ----  --------
  *           
              /wear    wear-service:8080 (172.17.0.4:8080)
              /watch   video-service:8080 (172.17.0.5:8080)
Annotations:  nginx.ingress.kubernetes.io/rewrite-target: /
              nginx.ingress.kubernetes.io/ssl-redirect: false
Events:
  Type    Reason  Age                From                      Message
  ----    ------  ----               ----                      -------
  Normal  Sync    11m (x2 over 11m)  nginx-ingress-controller  Scheduled for sync
```

Lets change our path to /stream instead of /watch

```
➜  kubectl edit ingress -n app-space ingress-wear-watch 
ingress.networking.k8s.io/ingress-wear-watch edited
```

```
➜  kubectl describe ingress -n app-space ingress-wear-watch 
Name:             ingress-wear-watch
Labels:           <none>
Namespace:        app-space
Address:          172.20.249.63
Ingress Class:    <none>
Default backend:  <default>
Rules:
  Host        Path  Backends
  ----        ----  --------
  *           
              /wear     wear-service:8080 (172.17.0.4:8080)
              /stream   video-service:8080 (172.17.0.5:8080)
Annotations:  nginx.ingress.kubernetes.io/rewrite-target: /
              nginx.ingress.kubernetes.io/ssl-redirect: false
Events:
  Type    Reason  Age               From                      Message
  ----    ------  ----              ----                      -------
  Normal  Sync    7s (x3 over 17m)  nginx-ingress-controller  Scheduled for sync
```

### Listing all the resources

```
➜  kubectl get ingress -n app-space
NAME                 CLASS    HOSTS   ADDRESS          PORTS   AGE
ingress-wear-watch   <none>   *       172.20.211.167   80      4m53s
```

➜  kubectl get deployments -n ingress-nginx
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
ingress-nginx-controller   1/1     1            1           6m12s


```
➜  kubectl get deployments -n app-space
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
default-backend   1/1     1            1           5m22s
webapp-video      1/1     1            1           5m22s
webapp-wear       1/1     1            1           5m22s
```

Lets list all resources in app-space namespace

```
➜  kubectl get all -n app-space
NAME                                   READY   STATUS    RESTARTS   AGE
pod/default-backend-569f95b877-nnl9t   1/1     Running   0          8m32s
pod/webapp-video-7d6646445c-4nvhl      1/1     Running   0          8m32s
pod/webapp-wear-7cf6df9954-pmkth       1/1     Running   0          8m32s

NAME                              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/default-backend-service   ClusterIP   172.20.22.115   <none>        80/TCP     8m32s
service/video-service             ClusterIP   172.20.15.162   <none>        8080/TCP   8m32s
service/wear-service              ClusterIP   172.20.155.35   <none>        8080/TCP   8m32s

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/default-backend   1/1     1            1           8m33s
deployment.apps/webapp-video      1/1     1            1           8m33s
deployment.apps/webapp-wear       1/1     1            1           8m33s

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/default-backend-569f95b877   1         1         1       8m33s
replicaset.apps/webapp-video-7d6646445c      1         1         1       8m33s
replicaset.apps/webapp-wear-7cf6df9954       1         1         1       8m33s
```

Lets list all resources in ingress-nginx namespace

```
➜  kubectl get all -n ingress-nginx
NAME                                            READY   STATUS      RESTARTS   AGE
pod/ingress-nginx-admission-create-rxh7b        0/1     Completed   0          9m17s
pod/ingress-nginx-admission-patch-ws26f         0/1     Completed   0          9m17s
pod/ingress-nginx-controller-6c4c749b95-d9qd9   1/1     Running     0          9m17s

NAME                                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx-controller             NodePort    172.20.211.167   <none>        80:30080/TCP,443:32103/TCP   9m17s
service/ingress-nginx-controller-admission   ClusterIP   172.20.155.111   <none>        443/TCP                      9m17s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-nginx-controller   1/1     1            1           9m17s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/ingress-nginx-controller-6c4c749b95   1         1         1       9m17s

NAME                                       STATUS     COMPLETIONS   DURATION   AGE
job.batch/ingress-nginx-admission-create   Complete   1/1           9s         9m17s
job.batch/ingress-nginx-admission-patch    Complete   1/1           9s         9m17s
```

### webapp-food deployment

```
✖ kubectl get deploy -n app-space
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
default-backend   1/1     1            1           19m
webapp-food       1/1     1            1           25s
webapp-video      1/1     1            1           19m
webapp-wear       1/1     1            1           19m
```

### Creating a new Path to /eat address

```
✖ kubectl edit ingress -n app-space ingress-wear-watch 
ingress.networking.k8s.io/ingress-wear-watch edited
```

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  creationTimestamp: "2025-03-08T14:01:15Z"
  generation: 3
  name: ingress-wear-watch
  namespace: app-space
  resourceVersion: "2956"
  uid: d6bbab6d-0228-4060-beb3-24f562d14e96
spec:
  rules:
  - http:
      paths:
      - backend:
          service:
            name: wear-service
            port:
              number: 8080
        path: /wear
        pathType: Prefix
      - backend:
          service:
            name: video-service
            port:
              number: 8080
        path: /stream
        pathType: Prefix
      - backend:
          service:
            name: food-service
            port:
              number: 8080
        path: /eat
        pathType: Prefix
status:
  loadBalancer:
    ingress:
    - ip: 172.20.249.63
```

## New deployment for webapp-pay


```
➜  kubectl get deploy -n critical-space
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
webapp-pay   1/1     1            1           92s
```


```
➜  kubectl get pods -n critical-space
NAME                          READY   STATUS    RESTARTS   AGE
webapp-pay-7df499586f-bcrrf   1/1     Running   0          30s
```

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  name: ingress-pay
  namespace: critical-space
spec:
  rules:
  - http:
      paths:
      - backend:
          service:
            name: pay-service
            port:
              number: 8282
        path: /pay
        pathType: Prefix
status:
  loadBalancer:
    ingress:
    - ip: 172.20.249.63
 ```

```
➜  kubectl apply -f ingress.yaml 
ingress.networking.k8s.io/ingress-pay created
```

```
➜  kubectl get svc -n critical-space
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
pay-service   ClusterIP   172.20.47.255   <none>        8282/TCP   9m11s
```


## Ingress-networking

```
➜  kubectl get deploy -A
NAMESPACE     NAME              READY   UP-TO-DATE   AVAILABLE   AGE
app-space     default-backend   1/1     1            1           4m42s
app-space     webapp-video      1/1     1            1           4m42s
app-space     webapp-wear       1/1     1            1           4m42s
```

```
➜  kubectl create ns ingress-nginx
namespace/ingress-nginx created
```

Lets create a configmap with no data

```
➜  kubectl create configmap ingress-nginx-controller -n ingress-nginx
configmap/ingress-nginx-controller created
```

