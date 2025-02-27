# Jobs

All pods have a default policy to restart with `restartPolicy` set to Always. We can set pod to not try to restart with `restartPolicy` set to `Never`:

```
apiVersion: v1
kind: Pod
metadata:
  name: throw-dice-pod
spec:
  containers:
  -  image: kodekloud/throw-dice
     name: throw-dice
  restartPolicy: Never
```

After running the pod with kubectl apply command we will see the pod with `Status` as `Completed` :

```
➜  kubectl get pods
NAME             READY   STATUS      RESTARTS   AGE
throw-dice-pod   0/1     Completed   0          4m16s
```

We can check the logs to investigate:

```
➜  kubectl logs throw-dice-pod 
6
```

## Creating a job yaml file

```
apiVersion: batch/v1
kind: Job
metadata:
  name: throw-dice-job
spec:
  backoffLimit: 15 # This is so the job does not quit before it succeeds.
  template:
    spec:
      containers:
      - image: kodekloud/throw-dice
        name: throw-dice
      restartPolicy: Never
```


```
➜  kubectl apply -f job-throw-dice.yaml 
job.batch/throw-dice-job created
```

This image will try restart the job until it gets number 6


```
➜  kubectl get pods
NAME                   READY   STATUS      RESTARTS   AGE
throw-dice-job-8cs8c   0/1     Error       0          84s
throw-dice-job-nnx69   0/1     Error       0          95s
throw-dice-job-tm5l2   0/1     Completed   0          22s
throw-dice-job-w4qtq   0/1     Error       0          63s
```


## Increasing the job to run twice


Given the yaml file we add `completions: 2` to it:

```
apiVersion: batch/v1
kind: Job
metadata:
  name: throw-dice-job
spec:
  completions: 2
  backoffLimit: 25 # This is so the job does not quit before it succeeds.
  template:
    spec:
      containers:
      - image: kodekloud/throw-dice
        name: throw-dice
      restartPolicy: Never
```

We apply the new job yaml file:

```
➜  kubectl apply -f job-throw-dice.yaml 
job.batch/throw-dice-job created
```


```
➜  kubectl get jobs
NAME             STATUS     COMPLETIONS   DURATION   AGE
throw-dice-job   Complete   2/2           3m58s      4m35s
```

Then we check how many time it had to run the pods until gets 2 times the number 6

```
➜  kubectl get pods
NAME                   READY   STATUS      RESTARTS   AGE
throw-dice-job-5nqq8   0/1     Error       0          4m32s
throw-dice-job-66xmh   0/1     Error       0          3m16s
throw-dice-job-8ldg6   0/1     Error       0          4m21s
throw-dice-job-kdgk9   0/1     Completed   0          3m19s
throw-dice-job-kxpx5   0/1     Error       0          2m43s
throw-dice-job-lg8vl   0/1     Error       0          2m2s
throw-dice-job-p5kqp   0/1     Error       0          4m
throw-dice-job-vmf6h   0/1     Error       0          3m4s
throw-dice-job-w59pg   0/1     Completed   0          41s
```

Notice that it created many pods and until gets 2 times the number 6

### Using parallelism to speed up the process - setting 3 pods at once

```
apiVersion: batch/v1
kind: Job
metadata:
  name: throw-dice-job
spec:
  completions: 2
  parallelism: 3
  backoffLimit: 25 # This is so the job does not quit before it succeeds.
  template:
    spec:
      containers:
      - image: kodekloud/throw-dice
        name: throw-dice
      restartPolicy: Never
```

```
➜  kubectl get jobs
NAME             STATUS    COMPLETIONS   DURATION   AGE
throw-dice-job   Running   2/2           89s        89s
```



```
➜  kubectl get pods
NAME                   READY   STATUS      RESTARTS   AGE
throw-dice-job-8f6gn   0/1     Error       0          102s
throw-dice-job-dbfmg   0/1     Completed   0          80s
throw-dice-job-fss55   0/1     Error       0          44s
throw-dice-job-lksh2   0/1     Error       0          65s
throw-dice-job-pfq7f   0/1     Completed   0          3s
throw-dice-job-qfjhq   0/1     Error       0          80s
throw-dice-job-qzrq5   0/1     Error       0          102s
throw-dice-job-tgjc4   0/1     Error       0          76s
```
