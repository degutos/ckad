# Persistent Volume and Persistent Volume Claim

In Kubernetes, **Persistent Volumes (PV)** and **Persistent Volume Claims (PVC)** are used to manage storage resources for applications running in pods. They are part of Kubernetes' storage management system, which allows for the decoupling of storage from the lifecycle of individual pods.

### **Persistent Volume (PV):**
- A **Persistent Volume (PV)** is a piece of storage in the Kubernetes cluster that has been provisioned by an administrator or dynamically provisioned using StorageClasses.
- PVs are independent of the lifecycle of any pod, meaning they exist beyond the pod's lifetime.
- PVs can represent different types of storage like NFS, cloud provider block storage (e.g., AWS EBS, GCE Persistent Disk), or even local storage.
- Once created, PVs are made available for use by applications running in the cluster.

### **Persistent Volume Claim (PVC):**
- A **Persistent Volume Claim (PVC)** is a request for storage by a pod. It specifies the desired size, access modes (e.g., ReadWriteOnce, ReadOnlyMany), and other attributes.
- A PVC allows a pod to "claim" a PV without needing to know the underlying storage details.
- When a PVC is created, Kubernetes looks for a suitable PV that matches the requested size and access modes, and then binds the PVC to that PV.

### **How PV and PVC Work Together:**
1. **Admin creates PVs** with a specific storage size and access mode.
2. A **user creates PVCs** specifying the storage requirements (size, access mode).
3. Kubernetes automatically binds a PVC to an available PV that matches the claim.
4. The **pod** that requested the PVC can now mount the volume, ensuring its data persists across pod restarts or rescheduling.

### Example:

- **Persistent Volume (PV) definition:**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data
```

- **Persistent Volume Claim (PVC) definition:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  resources:
    requests:
      storage: 5Gi
  accessModes:
    - ReadWriteOnce
```

In this example, the PVC would match the PV with the same size and access mode, allowing the pod to use the storage defined in the PV.

### Key Points:
- **PV** is a resource in the cluster representing physical storage.
- **PVC** is a request for storage by a pod.
- PVCs are automatically bound to available PVs by Kubernetes based on matching criteria.



Once you create a PVC use it in a POD definition file by specifying the PVC Claim name under persistentVolumeClaim section in the volumes section like this:


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```


The same is true for ReplicaSets or Deployments. Add this to the pod template section of a Deployment on ReplicaSet.



Reference URL: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes


# Volumes


```yaml
...
spec:
  containers:
    ...
    volumeMounts:
    - mountPath: /opt # /opt is a mount point in the container mounted /data which is a directory in the host
      name: data-volume

  volumes:
  - name: data-volume
    hostPath:
      path: /data # /data is directory created in the host
      type: Directory
```

## PV

```yaml
➜  cat pv.yaml 
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  capacity:
    storage: 100Mi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /pv/log
```

```bash
➜  kubectl apply -f pv.yaml 
persistentvolume/pv-log created
```

```yaml
➜  kubectl describe pv pv-log 
Name:            pv-log
Labels:          <none>
Annotations:     <none>
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    
Status:          Available
Claim:           
Reclaim Policy:  Retain
Access Modes:    RWX
VolumeMode:      Filesystem
Capacity:        100Mi
Node Affinity:   <none>
Message:         
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /pv/log
    HostPathType:  
Events:            <none>
```

## PVC

```yaml
➜  cat pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
```

```bash
➜  kubectl apply -f pvc.yaml
persistentvolumeclaim/claim-log-1 created

➜  kubectl get pvc 
NAME          STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
claim-log-1   Pending                  

➜  kubectl describe pvc claim-log-1 
Name:          claim-log-1
Namespace:     default
StorageClass:  
Status:        Pending
Volume:        
Labels:        <none>
Annotations:   <none>
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      
Access Modes:  
VolumeMode:    Filesystem
Used By:       <none>
Events:
  Type    Reason         Age               From                         Message
  ----    ------         ----              ----                         -------
  Normal  FailedBinding  7s (x7 over 88s)  persistentvolume-controller  no persistent volumes available for this claim and no storage class is set

➜  kubectl get pv
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pv-log   100Mi      RWX            Retain           Available                          <unset>                          8m38s


```
> [!IMPORTANT]
> As we see the `PVC` is in `Pending` state because it does not match the accessModes for the PV. Lets change the accessMode for this VPC.

```sh
➜  cat pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Mi

➜  kubectl apply -f pvc.yaml 
persistentvolumeclaim/claim-log-1 created

➜  kubectl get pvc
NAME          STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
claim-log-1   Bound    pv-log   100Mi      RWX                           <unset>                 16s

```
> [!NOTE]
> - Now we have the PVC created and in Bound state.


## Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  namespace: default
spec:
  containers:
  - env:
    - name: LOG_HANDLERS
      value: file
    image: kodekloud/event-simulator
    imagePullPolicy: Always
    name: event-simulator
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /log
      name: log-volume
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-4tjbq
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: log-volume
    persistentVolumeClaim: 
      claimName: claim-log-1

  - name: kube-api-access-4tjbq
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2025-03-15T15:28:35Z"
    status: "True"
    type: PodReadyToStartContainers
  - lastProbeTime: null
    lastTransitionTime: "2025-03-15T15:28:31Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2025-03-15T15:28:35Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2025-03-15T15:28:35Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2025-03-15T15:28:31Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://198da1742eaca36393af64552e36176328237cc03af1100f068f8293ff0890fd
    image: docker.io/kodekloud/event-simulator:latest
    imageID: docker.io/kodekloud/event-simulator@sha256:1e3e9c72136bbc76c96dd98f29c04f298c3ae241c7d44e2bf70bcc209b030bf9
    lastState: {}
    name: event-simulator
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2025-03-15T15:28:34Z"
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-4tjbq
      readOnly: true
      recursiveReadOnly: Disabled
  hostIP: 192.168.227.163
  hostIPs:
  - ip: 192.168.227.163
  phase: Running
  podIP: 172.17.0.4
  podIPs:
  - ip: 172.17.0.4
  qosClass: BestEffort
  startTime: "2025-03-15T15:28:31Z"
```

### Deleting PVC

```bash
➜  kubectl get pv pv-log 
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pv-log   100Mi      RWX            Retain           Bound    default/claim-log-1                  <unset>                          24m

➜  kubectl delete pvc claim-log-1 
persistentvolumeclaim "claim-log-1" deleted

✖ kubectl get pvc claim-log-1  ## The PVC is in Terminating state
NAME          STATUS        VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
claim-log-1   Terminating   pv-log   100Mi      RWX                           <unset>                 13m
```
> [!NOTE]
> Note: the PVC is in the Terminating state because it is used by the pod 

```bash
➜  kubectl get pods
NAME     READY   STATUS    RESTARTS   AGE
webapp   1/1     Running   0          4m41s

➜  kubectl get po,pvc
No resources found in default namespace.

➜  kubectl get pv
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM                 STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pv-log   100Mi      RWX            Retain           Released   default/claim-log-1                  <unset>                          30m

```
> [!Note]
> - Notice: When we delete the PVC the PV is in `Released` state



