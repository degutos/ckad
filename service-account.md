# Service account in kubernetes 

## Types of account 

- User (used by users or admins)
- Service (used by third part applications)


## Example of use of Service account 

- Jenkins 
- Prometheus
- Kubernetes Dashboard
- Any third part application who wants to access the kube-api to retrieve online information from the cluster.
  

### Usage of Service Account 

- When you have a third part application that need to access the kube-api you will need a token generated for the service account 
- When a service account is created its not going to create anymore a token for that customer account. 

## Creating a Service account 

```
 kubectl create serviceaccount dashboard-andre
serviceaccount/dashboard-andre created
```

```
kubectl get sa
NAME              SECRETS   AGE
dashboard-andre   0         11s
default           0         355d
```

Notice that we created a new service account. Every namespace for default has automatically created a default service account called "default" 


```
 kubectl describe sa dashboard-andre
Name:                dashboard-andre
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   <none>
Tokens:              <none>
Events:              <none>
```

Notice that when a Service account is created kubernetes doesn't create anymore a secret for that Service Account.





