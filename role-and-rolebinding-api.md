# API groups

```sh
$ curl https://kube-master:6443/version
```

```sh
$ curl https://kube-master:6443/api/v1/pods
```

we could also have other api's like:

- /metrics
- /healthz
- /version
- /api
- /apis
- /logs


## checking permissions for myself

```sh
$ kubectl auth can-i create deployment 

$ kubectl auth can-i delete nodes
```

## Authorization mode in kube-api 

```sh
 ✖ kubectl describe pod -n kube-system kube-apiserver-controlplane
Name:                 kube-apiserver-controlplane
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 controlplane/192.168.28.54
Start Time:           Tue, 25 Mar 2025 06:03:06 +0000
Labels:               component=kube-apiserver
                      tier=control-plane
Annotations:          kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.168.28.54:6443
                      kubernetes.io/config.hash: 24bc1be506a4de522fd35bbf9c93a6b9
                      kubernetes.io/config.mirror: 24bc1be506a4de522fd35bbf9c93a6b9
                      kubernetes.io/config.seen: 2025-03-25T06:02:44.890357248Z
                      kubernetes.io/config.source: file
Status:               Running
SeccompProfile:       RuntimeDefault
IP:                   192.168.28.54
IPs:
  IP:           192.168.28.54
Controlled By:  Node/controlplane
Containers:
  kube-apiserver:
    Container ID:  containerd://e1b67e98e9be8ffba927099e81ed7f6b0f918de5d4a2e59b38d15f13e6dbd476
    Image:         registry.k8s.io/kube-apiserver:v1.32.0
    Image ID:      registry.k8s.io/kube-apiserver@sha256:ebc0ce2d7e647dd97980ec338ad81496c111741ab4ad05e7c5d37539aaf7dc3b
    Port:          <none>
    Host Port:     <none>
    Command:
      kube-apiserver
      --advertise-address=192.168.28.54
      --allow-privileged=true
      --authorization-mode=Node,RBAC
      --client-ca-file=/etc/kubernetes/pki/ca.crt
      --enable-admission-plugins=NodeRestriction
      --enable-bootstrap-token-auth=true
      --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
      --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
      --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
      --etcd-servers=https://127.0.0.1:2379
      --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
      --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
      --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
      --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
      --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
      --requestheader-allowed-names=front-proxy-client
      --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
      --requestheader-extra-headers-prefix=X-Remote-Extra-
      --requestheader-group-headers=X-Remote-Group
      --requestheader-username-headers=X-Remote-User
      --secure-port=6443
      --service-account-issuer=https://kubernetes.default.svc.cluster.local
      --service-account-key-file=/etc/kubernetes/pki/sa.pub
      --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
      --service-cluster-ip-range=172.20.0.0/16
      --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
      --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    State:          Running
      Started:      Tue, 25 Mar 2025 06:02:58 +0000
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        250m
    Liveness:     http-get https://192.168.28.54:6443/livez delay=10s timeout=15s period=10s #success=1 #failure=8
    Readiness:    http-get https://192.168.28.54:6443/readyz delay=0s timeout=15s period=1s #success=1 #failure=3
    Startup:      http-get https://192.168.28.54:6443/livez delay=10s timeout=15s period=10s #success=1 #failure=24
    Environment:  <none>
    Mounts:
      /etc/ca-certificates from etc-ca-certificates (ro)
      /etc/kubernetes/pki from k8s-certs (ro)
      /etc/ssl/certs from ca-certs (ro)
      /usr/local/share/ca-certificates from usr-local-share-ca-certificates (ro)
      /usr/share/ca-certificates from usr-share-ca-certificates (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  ca-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/ssl/certs
    HostPathType:  DirectoryOrCreate
  etc-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/ca-certificates
    HostPathType:  DirectoryOrCreate
  k8s-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/pki
    HostPathType:  DirectoryOrCreate
  usr-local-share-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /usr/local/share/ca-certificates
    HostPathType:  DirectoryOrCreate
  usr-share-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /usr/share/ca-certificates
    HostPathType:  DirectoryOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute op=Exists
Events:
  Type     Reason                  Age    From     Message
  ----     ------                  ----   ----     -------
  Warning  FailedCreatePodSandBox  8m42s  kubelet  Failed to create pod sandbox: open /run/systemd/resolve/resolv.conf: no such file or directory
  Normal   Pulled                  8m29s  kubelet  Container image "registry.k8s.io/kube-apiserver:v1.32.0" already present on machine
  Normal   Created                 8m29s  kubelet  Created container: kube-apiserver
  Normal   Started                 8m29s  kubelet  Started container kube-apiserver

```

Let check the line for `--authorization-mode=Node,RBAC`



## Checking roles

```sh
➜  kubectl get roles 
No resources found in default namespace.
```

Lets check roles in all namespaces

```sh
➜  kubectl get roles -A
NAMESPACE     NAME                                             CREATED AT
blue          developer                                        2025-03-25T06:07:45Z
kube-public   kubeadm:bootstrap-signer-clusterinfo             2025-03-25T06:03:04Z
kube-public   system:controller:bootstrap-signer               2025-03-25T06:03:03Z
kube-system   extension-apiserver-authentication-reader        2025-03-25T06:03:03Z
kube-system   kube-proxy                                       2025-03-25T06:03:05Z
kube-system   kubeadm:kubelet-config                           2025-03-25T06:03:03Z
kube-system   kubeadm:nodes-kubeadm-config                     2025-03-25T06:03:03Z
kube-system   system::leader-locking-kube-controller-manager   2025-03-25T06:03:03Z
kube-system   system::leader-locking-kube-scheduler            2025-03-25T06:03:03Z
kube-system   system:controller:bootstrap-signer               2025-03-25T06:03:03Z
kube-system   system:controller:cloud-provider                 2025-03-25T06:03:03Z
kube-system   system:controller:token-cleaner                  2025-03-25T06:03:03Z
```

Lets count them

```sh
➜  kubectl get roles -A --no-headers | wc -l
12
```

Lets check one specific role `kube-proxy` and note that role allows `configmaps`  to `get` on `kube-proxy` resource 


```sh
➜  kubectl describe role  -n kube-system kube-proxy 
Name:         kube-proxy
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources   Non-Resource URLs  Resource Names  Verbs
  ---------   -----------------  --------------  -----
  configmaps  []                 [kube-proxy]    [get]
```



## Rolebinding 

```sh
✖ kubectl get rolebinding -n kube-system 
NAME                                                ROLE                                                  AGE
kube-proxy                                          Role/kube-proxy                                       24m
kubeadm:kubelet-config                              Role/kubeadm:kubelet-config                           24m
kubeadm:nodes-kubeadm-config                        Role/kubeadm:nodes-kubeadm-config                     24m
system::extension-apiserver-authentication-reader   Role/extension-apiserver-authentication-reader        24m
system::leader-locking-kube-controller-manager      Role/system::leader-locking-kube-controller-manager   24m
system::leader-locking-kube-scheduler               Role/system::leader-locking-kube-scheduler            24m
system:controller:bootstrap-signer                  Role/system:controller:bootstrap-signer               24m
system:controller:cloud-provider                    Role/system:controller:cloud-provider                 24m
system:controller:token-cleaner                     Role/system:controller:token-cleaner                  24m

```

Lets describe one rolebinding

```sh
➜  kubectl describe rolebinding -n kube-system kube-proxy 
Name:         kube-proxy
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  kube-proxy
Subjects:
  Kind   Name                                             Namespace
  ----   ----                                             ---------
  Group  system:bootstrappers:kubeadm:default-node-token  
```

we can see that the account that the kube-proxy role is assigned to is `bootstrappers:kubeadm:default-node-token  `


## checking permissions as a specific user

```sh
$ kubectl auth can-i list pods  --as dev-user
no
```

Lets Create the necessary roles and role bindings required for the dev-user to create, list and delete pods in the default namespace.


```sh
✖ kubectl create role developer --namespace=default --verb=list,create,delete --resource=pods
role.rbac.authorization.k8s.io/developer created
```

```sh
➜  kubectl create rolebinding dev-user-binding --namespace=default --role=developer --use
r=dev-user
rolebinding.rbac.authorization.k8s.io/dev-user-binding created
```

A set of new roles and role-bindings are created in the blue namespace for the dev-user. However, the dev-user is unable to get details of the dark-blue-app pod in the blue namespace. Investigate and fix the issue.

```sh
➜  kubectl edit role developer -n blue
role.rbac.authorization.k8s.io/developer edited
```

all we needed to do is add the right resourceNames:

```sh
  resourceNames:
  - dark-blue-app
```
