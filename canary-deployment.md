# Canary deployment

Canary deployment is a way of test your new deployment without remove the old deployment.


Notice we have only 1 deployment called `frontend` 

```
➜  kubectl get deploy
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
frontend   5/5     5            5           117s
```

Notice that this deployment has a `selector` app=frontend

```
 ➜  kubectl describe deploy frontend
Name:                   frontend
Namespace:              default
CreationTimestamp:      Wed, 26 Feb 2025 19:13:06 +0000
Labels:                 app=myapp
                        tier=frontend
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=frontend
Replicas:               5 desired | 5 updated | 5 total | 5 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=frontend
           version=v1
  Containers:
   webapp-color:
    Image:         kodekloud/webapp-color:v1
    Port:          <none>
    Host Port:     <none>
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   frontend-8b5db9bd (5/5 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  3m4s  deployment-controller  Scaled up replica set frontend-8b5db9bd from 0 to 5
```

Notice we have a service called `frontend-service`

```
➜  kubectl get svc
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
frontend-service   NodePort    172.20.242.38   <none>        8080:30080/TCP   4m50s
```

## Canary

Notice we have 2 deployments now, `frontend` and `frontend-v2`

```
 ➜  kubectl get deploy 
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
frontend      5/5     5            5           6m34s
frontend-v2   2/2     2            2           21s
```

Lets decrease the number of replicas of the deployment `frontend-v2`

```
➜  kubectl scale deploy frontend-v2  --replicas=1
deployment.apps/frontend-v2 scaled

➜  kubectl get deploy
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
frontend      5/5     5            5           9m8s
frontend-v2   1/1     1            1           2m55s
```

## Redirecting users to frontend-v2

Lets scale down the frontend deployment

```
➜  kubectl scale deploy frontend --replicas=0
deployment.apps/frontend scaled

➜  kubectl get deploy
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
frontend      0/0     0            0           11m
frontend-v2   1/1     1            1           5m42s
```


Now lets scale up frontend-v2 to 5 replicas

```
➜  kubectl scale deploy frontend-v2 --replicas=5
deployment.apps/frontend-v2 scaled

➜  kubectl get deploy
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
frontend      0/0     0            0           13m
frontend-v2   5/5     5            5           6m58s
```

## Deleting the old deployment

```
➜  kubectl delete deploy frontend
deployment.apps "frontend" deleted

controlplane ~ ➜  kubectl get deploy
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
frontend-v2   5/5     5            5           8m2s
```

