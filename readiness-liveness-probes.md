# Readiness Probe

A readiness probe in Kubernetes is a mechanism that allows the system to check whether a container is ready to handle requests. It is used to determine when a container has completed its startup process and is ready to serve traffic.

When you configure a readiness probe, Kubernetes will periodically check the state of the container using a configured method (such as HTTP requests, TCP socket checks, or executing commands inside the container). If the container is not ready, Kubernetes will stop sending traffic to it, allowing time for the container to initialize properly.

Key Points:

Purpose: Determines if a container can handle traffic (i.e., if it's "ready").
When it's used: Typically, readiness probes are helpful when your application needs time to start or initialize before it can serve requests (e.g., waiting for a database to be available or completing a time-consuming initialization).
Behavior:
If the readiness probe fails, the container is considered not ready, and Kubernetes will remove it from the load balancer's pool of available endpoints, preventing traffic from being sent to it.
Once the probe starts succeeding, the container is considered ready again, and traffic will be routed to it.
Types of Readiness Probes:
HTTP Request: Kubernetes sends an HTTP GET request to a specific URL within the container. If the HTTP status code is 2xx or 3xx, the container is considered ready.

TCP Socket: Kubernetes attempts to open a TCP connection to a specified port inside the container. If the connection is successful, the container is marked as ready.

Command: Kubernetes runs a command inside the container. If the command exits with a status code of 0, the container is considered ready.

Example Configuration:

```
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    image: example-image
    readinessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 5  # Time to wait before performing the first probe
      periodSeconds: 10       # How often to perform the probe
      failureThreshold: 3     # Number of consecutive failures before marking as not ready
```

Benefits of Using Readiness Probes:

Ensures that only fully initialized containers receive traffic.
Helps manage situations where containers need to wait for external services (like databases) to be ready.
Prevents downtime or failed requests due to containers that aren't ready yet.
In contrast to a liveness probe, which checks if the container is still alive (and might restart it if it's not), the readiness probe focuses on the container's ability to handle traffic at all.



## Script to test Readiness Probe

```
cat curl-test.sh 
for i in {1..20}; do
   kubectl exec --namespace=kube-public curl -- sh -c 'test=`wget -qO- -T 2  http://webapp-service.default.svc.cluster.local:8080/ready 2>&1` && echo "$test OK" || echo "Failed"';
   echo ""
done
```

