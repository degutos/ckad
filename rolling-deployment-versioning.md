# Rolling deployment

In Kubernetes, a **rolling deployment** is a deployment strategy that gradually updates the Pods in a Deployment with a new version of an application, without causing downtime. This is achieved by updating the Pods one at a time (or in batches) while ensuring that the desired number of Pods are always running and available to serve traffic.

Here’s a breakdown of how a rolling deployment works in Kubernetes:

1. **New Version of the Application**: A new version of your application (e.g., a new container image) is pushed to the Kubernetes cluster.

2. **Gradual Updates**: Kubernetes updates the Pods in the Deployment one by one or in small batches. It terminates old Pods and creates new ones with the updated application version.

3. **Zero Downtime**: Kubernetes ensures that the number of running Pods remains consistent during the deployment. This means there will always be a set of Pods running to handle traffic, even as the application is being updated. The rolling update is done in a way that minimizes the chances of downtime.

4. **Health Checks**: Kubernetes uses health probes (liveness and readiness probes) to ensure that the new Pods are ready to take traffic before terminating the old ones. If a new Pod fails its health check, it won’t receive traffic, and the deployment is paused or rolled back.

5. **Rollback**: If something goes wrong with the new version, Kubernetes can automatically roll back to the previous version of the deployment to ensure service continuity.

### Key Components:
- **Deployment**: The Kubernetes object that manages the rolling update process.
- **Pods**: The smallest deployable units, which are updated in batches.
- **ReplicaSet**: A group of Pods maintained by the Deployment. It ensures the correct number of replicas are running.

### Benefits:
- **No downtime**: The service is still available while the update is in progress.
- **Rollback capability**: Kubernetes can roll back to a previous stable version if any issues occur during the deployment.
- **Controlled Update**: You can control how many Pods are updated at a time, thus minimizing the impact on users.

You can control the rolling deployment behavior by specifying parameters like:
- **maxSurge**: The maximum number of Pods that can be created over the desired number of Pods during the update.
- **maxUnavailable**: The maximum number of Pods that can be unavailable during the update.


## kubectl rollout


We can see the status of deployment by running:

```
kubectl rollout status deployment/myapp-deployment
```


To see their history and revisions:

```
kubectl rollout history deployment/myapp-deployment
```

## Deployment strategy

-**Recreate**: This strategy is going to put down all pods for that deployment and then spin up all pods with new images/revisions. This strategy has a `Down Time`  application, and this is not the default kubernetes deploy strategy

-**Rolling Update**: This is the default kubernetes deploy strategy. It put down one pod with old version and put up a new pod with new image, after the first pod is upgraded it moves to second pod and recycle it one by one until the last pod updated. There is no application down time on this strategy.

## Rollback version

If we did a deployment to a new version and we want go back to previously version we run:

```
kubectl rollout undo deploy/myapp-deployment
```

We will notice at least 02 replicasets demonstrating these two deployments and number of pods for each replicasets. 

## Deployment restart

If we want just restart a deployment to recycle pods:

```
kubectl rollout restart deployment myapp-deployment -n <namespace>
```



## Summarize commands:


To create a deploy


```
kubectl create -f deployment-definition.yaml
```

To get a deployment 

```
kubectl get deployments
```

To update a deployment

```
kubectl apply -f deployment-definition.yaml
```


```
kubectl set image deployment/myapp-deployment nginx:nginx:1.9.1
```

To get Status of deployment

```
kubectl rollout status deployment/myapp-deployment
```

```
kubectl rollout history deployment/myapp-deployment
```

To Rollback a deployment version/image:

```
kubectl rollout undo deployment/myapp-deployment
```

```
➜  ~ kubectl create deployment nginx --image=nginx:1.16
deployment.apps/nginx created

➜  ~ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-599c4f6679-tbltw   1/1     Running   0          107s
simple-webapp-1          1/1     Running   0          3d11h

➜  ~ kubectl rollout status deployment nginx
deployment "nginx" successfully rolled out

➜  ~ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>

---

➜  ~ kubectl create deploy nginx --image=nginx:1.16
deployment.apps/nginx created

➜  ~ kubectl get deploy
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           6s

➜  ~ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-599c4f6679-j25c5   1/1     Running   0          20s
simple-webapp-1          1/1     Running   0          3d11h

kubectl rollout status deployment nginx
deployment "nginx" successfully rolled out

➜  ~ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>


➜  ~ kubectl rollout history deployment nginx --revision=1
deployment.apps/nginx with revision #1
Pod Template:
  Labels:	app=nginx
	pod-template-hash=599c4f6679
  Containers:
   nginx:
    Image:	nginx:1.16
    Port:	<none>
    Host Port:	<none>
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
  Node-Selectors:	<none>
  Tolerations:	<none>


  ➜  ~ kubectl set image deployment nginx nginx=nginx:1.17 --record
Flag --record has been deprecated, --record will be removed in the future
deployment.apps/nginx image updated


➜  ~ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment nginx nginx=nginx:1.17 --record=true

➜  ~ kubectl rollout undo deployment nginx --to-revision=1
deployment.apps/nginx rolled back

  ~ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
2         kubectl set image deployment nginx nginx=nginx:1.17 --record=true
3         kubectl edit deployment nginx --record=true
4         <none>
```

## Curl command to test communication to the pod http service

```
 ➜  cat curl-test.sh 
for i in {1..3}; do
   kubectl exec --namespace=kube-public curl -- sh -c 'test=`wget -qO- -T 2  http://webapp-service.default.svc.cluster.local:8080/info 2>&1` && echo "$test OK" || echo "Failed"';
   echo ""
done
```

- The output would be something like this:

```
~ ➜  ./curl-test.sh 
Hello, Application Version: v1 ; Color: blue OK

Hello, Application Version: v1 ; Color: blue OK

Hello, Application Version: v1 ; Color: blue OK
```