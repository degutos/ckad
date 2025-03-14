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


## Listing a Serivce account 

```
controlplane ~ ➜  kubectl get serviceaccount
NAME      SECRETS   AGE
default   0         8m53s
dev       0         28s
```

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

### Creating a token

```
kubectl create token dashboard-andre
eyJhbGciOiJSUzI1NiIsImtpZCI6IldNZDJPR2dHVUs3YXR6ZWJUZnkzb2pRdVo1MWJZVjdyMm1xMlRLMWFrX00ifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzM4MzI3NDQ3LCJpYXQiOjE3MzgzMjM4NDcsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0Iiwic2VydmljZWFjY291bnQiOnsibmFtZSI6ImRhc2hib2FyZC1hbmRyZSIsInVpZCI6ImQxYzc2ODA2LTBhM2ItNDUxNS1iNjg4LWEwM2JmZDA0ODM3MCJ9fSwibmJmIjoxNzM4MzIzODQ3LCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpkYXNoYm9hcmQtYW5kcmUifQ.DjUhUI4L2vFlbYeebaz578LLeTZtJZcWlHIZbE9IAoFwpbAujX-v6rLStPV7BrDceBNiCz1Hca-qiU6vC_f6XVwQBX5NL9_XPbp1Ily5cv_oHZ7ktGjEsgs33dooVsPguMkrQcUB2LwhT3-kU21yqy-k3YDQ2VmgAPwozj5UYTlPOGtHreqBTEHyhTOBUptLwJlx1jD2QhLUlTiv4cG65JOWWlp2J3sNo3ksPVA-stLXFZCUTK5ACwtZxyMFu4ARv49-DwRXUwTi-1GqRdAzsh9kM7qe1HaVbU1mDNJZSfh9HKktIjJmMoQ79Dne5tYkJqxk-skTqCbvwh3LfUZDLg
```

### Creating a secret for a service account 

```
 $ cat secret-service-account.yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: my-secret
  annotations:
    kubernetes.io/service-account.name: dashboard-andre
```

```
kubectl apply -f secret-service-account.yaml
secret/my-secret created
```

```
kubectl get secrets my-secret -o yaml
apiVersion: v1
data:
  ca.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJQ1pwU3lyalEzcUV3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TkRBeU1UQXhORE0zTkRaYUZ3MHpOREF5TURjeE5EUXlORFphTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUUNza3U1bDBNYllmVjA5Ulp2emNVRU0vMzJwK0YvdFU2cStkeTU0NUREWjdnZDJIRzVEUWFQZGNJcXYKRjdnVHhXbVJvbkt5SE0rZ3Y2UVpCbStvbmUvQ2VaYy9QVG5uTFdmWjllYi9EUzlKSitXSmlDRm9TS2p1cDVmTQoxL3BpUmw2YVFoeFdYdzdiS2JzOG4ydUlJTlE3cEZuclVkRE9iSnNFNUJnUitZNDVRVFlMNWdCK093MWFDc2FRCnVFYVpNTEprVmNaSkVNRHUvNUxqbHhyakhwNXJraHVWZ1R3d3I0OGF5clRuMTN0T01pMlI5U1F5SFdjQjVRVFUKTWQyN2IxdUxMOFh4UkFKaW9lU3FVZVhrcytvNFJ0NTZCbktRL3F4NUVXdkZlekRZa2xBRWR3djZjdGRMcEs0NQpUTlNKY05YWnVvbWM3MjV3N2JqUWFFTkx5RDliQWdNQkFBR2pXVEJYTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQCkJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJSSVhBNUs3Ujc4S1hVOFZTMWYvWEtUSm95TVdEQVYKQmdOVkhSRUVEakFNZ2dwcmRXSmxjbTVsZEdWek1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQk1KS3c3ZHdHOApJQjkzeUI4dGNydHBpMS93Mm82V3N3OHFJd2p4dlp1bDczZHlyR0VubjFrenpZZmtDbjZCZ2dZMytpZXIxUjRnClNVSXgyY2pwSlQ4Sm01ZVBKMnVoN2hxdnBDUmRRQmZwTDJmWUJzS3NCdFZveG9OYW9McU15dUI2VFJlNnBnYzcKS3V1YlZpK3d1QkdrVnl0eEJza09hUzZBOUVId3ZhcEpaaDh2dkZyNEJqeTZ4VFBtS2FJY2JOREFscUk2S0daLwovTDFZSWtaV0wwS2hQNnNnbll4U3FzNXVDaVBkWjBNa3BvUUNVck1wSWtpeU43ZU02K3RSakpoRVhBc0czblFKCmJRdXE2dGs3a3NlTDhHMXhIRlpqVXowNGI5WlRJTXFORXJPRUY4RjB6b0RsU0RVRHp1UG1ZNGNyaUMxYUdwdEgKcm9td3NScXdta3JwCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
  namespace: ZGVmYXVsdA==
  token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNklsZE5aREpQUjJkSFZVczNZWFI2WldKVVpua3piMnBSZFZvMU1XSlpWamR5TW0xeE1sUkxNV0ZyWDAwaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUprWldaaGRXeDBJaXdpYTNWaVpYSnVaWFJsY3k1cGJ5OXpaWEoyYVdObFlXTmpiM1Z1ZEM5elpXTnlaWFF1Ym1GdFpTSTZJbTE1TFhObFkzSmxkQ0lzSW10MVltVnlibVYwWlhNdWFXOHZjMlZ5ZG1salpXRmpZMjkxYm5RdmMyVnlkbWxqWlMxaFkyTnZkVzUwTG01aGJXVWlPaUprWVhOb1ltOWhjbVF0WVc1a2NtVWlMQ0pyZFdKbGNtNWxkR1Z6TG1sdkwzTmxjblpwWTJWaFkyTnZkVzUwTDNObGNuWnBZMlV0WVdOamIzVnVkQzUxYVdRaU9pSmtNV00zTmpnd05pMHdZVE5pTFRRMU1UVXRZalk0T0MxaE1ETmlabVF3TkRnek56QWlMQ0p6ZFdJaU9pSnplWE4wWlcwNmMyVnlkbWxqWldGalkyOTFiblE2WkdWbVlYVnNkRHBrWVhOb1ltOWhjbVF0WVc1a2NtVWlmUS5mRzBvWG5rYUhIU0c4WUxGaC1tb0FkaXBQMk0yT01ETGY2X1Uzak1VMXhXVk1zU0tpVnNOVkw3ZDhLTnEwT3g3ODhVMjd1a09KdXQ1NkpJT01zNlp6RVN3dVRqUWFZTzVXUWxfS040aGVkc3lXOUhYcTVZVkNYUmlUdkFJLXI1WTBUNEJ1VUxhbjRiLTJGbEZWcDVHcVBZU193YVd0MFlkdndNeW9jOGlvMXE3dDZucEU2UXA5QUM1RzdEMTE2VVJqVExGMWZQeUlpQkswNjFmVkpiV1ZBc05uejA3Rjl2S01WeXRMenNlanFYejFCMnhvNFVETncwUGg5VmVjRFoteVdLcmdDR3FUaXZXb3QwSkw1dWxHaC1HSUNsTTVLQ2lGaGRjUnVpeWl3VEVhUUhGakpBTzRzb2I0bUJmajVQS21WQ3NfY2xVblNTTXctZ3ZSSmVxRGc=
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Secret","metadata":{"annotations":{"kubernetes.io/service-account.name":"dashboard-andre"},"name":"my-secret","namespace":"default"},"type":"kubernetes.io/service-account-token"}
    kubernetes.io/service-account.name: dashboard-andre
    kubernetes.io/service-account.uid: d1c76806-0a3b-4515-b688-a03bfd048370
  creationTimestamp: "2025-01-31T11:52:12Z"
  name: my-secret
  namespace: default
  resourceVersion: "2473435"
  uid: 661d3b68-dc30-4ba9-bf11-8be1a2e2d3f2
type: kubernetes.io/service-account-token
```

### Creating a pod to use the secret

```
kubectl run my-pod --image=nginx -o yaml --dry-run=client > my-pod.yaml
```

Add the serviceAccountName parameter to your pod yaml file

```
spec:
  serviceAccountName: dashboard-andre
  containers:
```

For example:

```
cat my-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: my-pod
  name: my-pod
spec:
  serviceAccountName: dashboard-andre
  containers:
  - image: nginx
    name: my-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```
kubectl apply -f my-pod.yaml
pod/my-pod created
```

```
 kubectl get pod my-pod -o yaml | grep serviceAccount
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"creationTimestamp":null,"labels":{"run":"my-pod"},"name":"my-pod","namespace":"default"},"spec":{"containers":[{"image":"nginx","name":"my-pod","resources":{}}],"dnsPolicy":"ClusterFirst","restartPolicy":"Always","serviceAccountName":"dashboard-andre"},"status":{}}
  serviceAccount: dashboard-andre
  serviceAccountName: dashboard-andre
  ```




