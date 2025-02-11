# Init Container

In Kubernetes, an initContainer is a special type of container that runs before the main containers in a pod start. Init containers allow you to run initialization tasks such as setup, configuration, or checking dependencies before the main application containers begin their execution.

Key characteristics of initContainers:

Run before main containers: Init containers always run sequentially (one after the other) before the primary containers of the pod start. They are executed in the order they are defined.

Can be used for setup tasks: Init containers can be used for tasks like downloading configuration files, setting up database schemas, or waiting for some dependency to be available.

Exit successfully before main containers start: If an init container fails (non-zero exit code), the pod will not start the main containers, and the init container will be retried according to the pod’s restart policy.

Can have different images: Init containers can run with different images and configurations compared to the main containers, allowing them to be more specialized.

Resource limitations: Init containers have resource limits and requests like any regular container, which can be configured individually.


```
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  initContainers:
  - name: init-db
    image: busybox
    command: ["sh", "-c", "echo 'Initializing DB' && sleep 5"]
  containers:
  - name: main-app
    image: my-app-image
    ports:
    - containerPort: 80

```

Init containers are useful for ensuring that certain steps are completed before the main application starts, providing a flexible way to manage startup dependencies and tasks.

```
---
apiVersion: v1
kind: Pod
metadata:
  name: red
  namespace: default
spec:
  containers:
  - command:
    - sh
    - -c
    - echo The app is running! && sleep 3600
    image: busybox:1.28
    name: red-container
  initContainers:
  - image: busybox
    name: red-initcontainer
    command: 
      - "sleep"
      - "20"
```


## Example of initContainer

```
  initContainers:
  - command:
    - sh
    - -c
    - sleep 2;
    image: busybox
    name: init-myservice
```

```
  - command:
    - sh
    - -c
    - echo The app is running! && sleep 3600
    image: busybox:1.28
    name: orange-container
```

