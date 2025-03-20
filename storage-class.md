# Storage Class

```sh
➜  kubectl get sc
NAME                        PROVISIONER                     RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-path (default)        rancher.io/local-path           Delete          WaitForFirstConsumer   false                  12m
local-storage               kubernetes.io/no-provisioner    Delete          WaitForFirstConsumer   false                  6m35s
portworx-io-priority-high   kubernetes.io/portworx-volume   Delete          Immediate              false                  6m35s
```

## Describing storage class

```yaml
~ ➜  kubectl describe sc local-storage
Name:            local-storage
IsDefaultClass:  No
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{},"name":"local-storage"},"provisioner":"kubernetes.io/no-provisioner","volumeBindingMode":"WaitForFirstConsumer"}

Provisioner:           kubernetes.io/no-provisioner
Parameters:            <none>
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     WaitForFirstConsumer
Events:                <none>
```

```yaml
➜  kubectl describe sc local-path
Name:                  local-path
IsDefaultClass:        Yes
Annotations:           defaultVolumeType=local,objectset.rio.cattle.io/applied=H4sIAAAAAAAA/4yRz47UMAyHXwX53JYpnamqSBxg0V4QEhJoObuJOzVN4ypxi0areXeUMqDhwJ9j8ov9xZ+fARd+ophYAhhIKhHPVE1dqlhebjUUMHFwYODTj+jBY0pQwEyKDhXBPAOGIIrKElI+Ohpw9fokfp3p82UhMODFoocCpP9KVhNpFVkqi6qeMokz4i+5fAsUy/M2gYGpSXfJVhcv3nNwr984J+GfLQLOv/5T3sb9r6K0oM2V09pTmS5JaYbipzCbrVQ5ioGUdnmcypuJco/BgMaV4FqAx5787upP3BHTCAbqrhmak21Pw9Db5tAe20MzHJuhPnUH19m2w1cOe3fMTX+bbEEd8+USZeO8XIpgIGKwI8UMuHtWQMwD8PxRPNsLGHhHnjRr2fYdvuXgOJw/iMuAL8j6KPGRY9IHCWmdKcL1ewAAAP//KQ1Ko0kCAAA,objectset.rio.cattle.io/id=,objectset.rio.cattle.io/owner-gvk=k3s.cattle.io/v1, Kind=Addon,objectset.rio.cattle.io/owner-name=local-storage,objectset.rio.cattle.io/owner-namespace=kube-system,storageclass.kubernetes.io/is-default-class=true
Provisioner:           rancher.io/local-path
Parameters:            <none>
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     WaitForFirstConsumer
Events:                <none>

➜  kubectl describe sc portworx-io-priority-high 
Name:            portworx-io-priority-high
IsDefaultClass:  No
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{},"name":"portworx-io-priority-high"},"parameters":{"priority_io":"high","repl":"1","snap_interval":"70"},"provisioner":"kubernetes.io/portworx-volume"}

Provisioner:           kubernetes.io/portworx-volume
Parameters:            priority_io=high,repl=1,snap_interval=70
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     Immediate
Events:                <none>
```

## PV

```sh
➜  kubectl get pv
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS    VOLUMEATTRIBUTESCLASS   REASON   AGE
local-pv   500Mi      RWO            Retain           Available           local-storage   <unset>                          11m

➜  kubectl describe pv local-pv
Name:              local-pv
Labels:            <none>
Annotations:       <none>
Finalizers:        [kubernetes.io/pv-protection]
StorageClass:      local-storage
Status:            Available
Claim:             
Reclaim Policy:    Retain
Access Modes:      RWO
VolumeMode:        Filesystem
Capacity:          500Mi
Node Affinity:     
  Required Terms:  
    Term 0:        kubernetes.io/hostname in [controlplane]
Message:           
Source:
    Type:  LocalVolume (a persistent volume backed by local storage on a node)
    Path:  /opt/vol1
Events:    <none>
```

## Creating a PVC

This pvc should be attached to the above local-pv

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc 
spec:
  storageClassName: local-storage 
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

