# Commands in docker


Pods will die after tasks is complete

```
$ docker run ubuntu
```
After running this the pod will die because it runs /bin/bash and closes it 

```
docker ps 
```
This will show no container running

```
docker ps -a 
```
We can see container ran and exited afterwards 


## Docker Manifest file

Lets see how we set a container to run a specific program as soon and the container gets running

```
CMD [ "nginx" ]
```

or 

```
CMD ["mysqld"]
```

or

```
CMD ["bash"]
```


Lets see how to set a container ubuntu to wait 5 seconds before exiting 
```
docker run ubuntu sleep 5
```

```
FROM ubuntu
CMD sleep 5
```

or 

```
From ubuntu
CMD ["sleep","5"]
```

Lets build our container image
```
docker build -t ubuntu-sleeper .
```

Then we run the docker command to create it 
```
docker run ubuntu-sleeper
```

Or we run ubuntu-sleeper and set to sleep 10s
```
docker run ubuntu-sleeper sleep 10
```

How do set the sleep command to not be mandotory during the run container
```
From ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

```
docker ubuntu-sleeper 
```

or

```
docker ubuntu-sleeper 10
```

How do we run a different program
```
docker run --entrypoint sleep-2.0 10
```

## Commands and Arguments for Pods

docker run --name ubuntu-sleeper ubuntu-sleeper

docker run --name ubuntu-sleeper ubuntu-sleeper 10


Lets see how we create a pod-definition.yaml file

```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      args: ["10"]
      command: ["sleep"]
```

As we see above the `args` parameter relates to `CMD` parameter in docker world 
and the `command` parameter relates to `ENTRYPOINT` in docker world


Example of pod hardcoded with sleep 5000

```
apiVersion: v1
kind: Pod 
metadata:
  name: ubuntu-sleeper-2
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep","5000"]
```

OR
```
apiVersion: v1
kind: Pod 
metadata:
  name: ubuntu-sleeper-2
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep"]
    args: ["5000"]
```


Example 2:

```
apiVersion: v1
kind: Pod 
metadata:
  name: ubuntu-sleeper-3
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command:
      - "sleep"
      - "1200"
```

Docker file example:

```
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]
```


## How to create a ConfigMap

```
controlplane ~ ➜  k create cm webapp-config-map --from-literal=APP_COLOR=darkblue
configmap/webapp-config-map created

```

Lets now change our pod to use that CM

```
...
spec:
  containers:
    - envFrom:
      - configMapRef:
          name: webapp-config-map
    image: kodekloud/webapp-color
...
```







