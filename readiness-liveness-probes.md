# Readiness Probe

A readiness probe in Kubernetes is a mechanism that allows the system to check whether a container is ready to handle requests. It is used to determine when a container has completed its startup process and is ready to serve traffic.

When you configure a readiness probe, Kubernetes will periodically check the state of the container using a configured method (such as HTTP requests, TCP socket checks, or executing commands inside the container). If the container is not ready, Kubernetes will stop sending traffic to it, allowing time for the container to initialize properly.

Key Points:

Purpose: Determines if a container can handle traffic (i.e., if it's "ready").
When it's used: Typically, readiness probes are helpful when your application needs time to start or initialize before it can serve requests (e.g., waiting for  a database to be available or completing a time-consuming initialization).
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

Another way of testing http connection to an application:

```
➜  ~ for i in {1..10} ; do wget -qO- -T 2 https://30080-port-4mjg73opx26ilaka.labs.kodekloud.com/readyy 2>&1 && echo " - Test OK" || echo " - FAILED" ; done
```

# Liveness Probe

A **Liveness Probe** in Kubernetes is a mechanism to monitor the health of a container running in a pod. It helps Kubernetes determine whether a container is still working properly. If the container fails the liveness probe, Kubernetes can automatically restart the container to attempt recovery.

### Purpose of a Liveness Probe:
- **Ensure the application is running:** If the application inside the container becomes unresponsive or stuck, the liveness probe detects this and triggers a restart.
- **Automatic recovery:** If a container becomes unhealthy, Kubernetes restarts it automatically, preventing downtime or errors from affecting the application for long periods.

### How Liveness Probes Work:
- **Check health status:** Kubernetes periodically sends requests to the container to check if it's alive (responsive).
- **Restart on failure:** If the container fails the probe a certain number of times (defined by the probe's settings), Kubernetes will restart the container.

### Types of Liveness Probes:
Kubernetes supports three types of checks to determine the health of a container:
1. **HTTP GET Probe:** Kubernetes sends an HTTP GET request to a specific endpoint on the container to check its health.
2. **TCP Socket Probe:** Kubernetes tries to open a TCP connection to a specified port on the container.
3. **Command Probe:** Kubernetes runs a command inside the container and checks the exit status to determine if the container is alive.

### Liveness Probe Configuration Parameters:
You can configure several parameters for a liveness probe:

- **`initialDelaySeconds`**: The number of seconds to wait before performing the first probe after the container starts.
- **`periodSeconds`**: How often the probe should run (in seconds).
- **`timeoutSeconds`**: The number of seconds to wait for a response before considering the probe failed.
- **`failureThreshold`**: The number of failed probes before the container is restarted.
- **`successThreshold`**: The number of consecutive successful probes before considering the container healthy.
  
### Example of Liveness Probe in a Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
    - name: my-container
      image: my-image
      livenessProbe:
        httpGet:
          path: /healthz
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 10
        timeoutSeconds: 2
        failureThreshold: 3
```

In this example:
- Kubernetes will send an HTTP GET request to `/healthz` on port `8080`.
- The probe will start after an initial delay of 5 seconds (`initialDelaySeconds: 5`).
- The probe will run every 10 seconds (`periodSeconds: 10`).
- If the probe takes more than 2 seconds to respond, it will be considered failed (`timeoutSeconds: 2`).
- If the probe fails 3 times in a row (`failureThreshold: 3`), the container will be restarted.

### Why use a Liveness Probe?
- **Prevent stale or stuck containers:** It ensures that your application doesn’t run indefinitely in an unhealthy state.
- **Self-healing:** When a container fails, Kubernetes will automatically restart it, improving application reliability.
- **Configurable behavior:** You can customize when and how probes are performed, making it easier to fit into your application's lifecycle and needs.

### Key Takeaways:
- Liveness probes are crucial for ensuring the health and stability of containers in production.
- They are configurable and can use HTTP, TCP, or commands to check health.
- Kubernetes automatically restarts the container if the liveness probe fails, minimizing downtime and ensuring application availability.