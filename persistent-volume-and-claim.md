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


```
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





