# Encrypting Secret Data at Rest in etcd

Reference: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/


#### Make sure you have etcd pod running  and you have etcdctl app installed

```
controlplane ~ ➜  k get pods -A
NAMESPACE     NAME                                   READY   STATUS    RESTARTS      AGE
default       webapp-color                           1/1     Running   0             6m44s
e-commerce    e-com-1123                             1/1     Running   0             6m44s
kube-system   coredns-787d4945fb-pdw7l               1/1     Running   0             75m
kube-system   coredns-787d4945fb-zlqg5               1/1     Running   0             75m
kube-system   etcd-controlplane                      1/1     Running   0             75m
kube-system   kube-apiserver-controlplane            1/1     Running   0             75m
kube-system   kube-controller-manager-controlplane   1/1     Running   0             75m
kube-system   kube-proxy-4xfkp                       1/1     Running   0             75m
kube-system   kube-scheduler-controlplane            1/1     Running   0             75m
kube-system   weave-net-7z75h                        2/2     Running   1 (75m ago)   75m
marketing     redis-67b8d598bc-vvwlc                 1/1     Running   0             6m44s
```


```
controlplane ~ ➜  etcdctl
-bash: etcdctl: command not found
```


Confirm your linux distro

```
controlplane ~ ➜  cat /etc/os-release 
NAME="Ubuntu"
VERSION="20.04.5 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.5 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal
```

##### Installing etcd-client

```
controlplane ~ ✖ apt-get install etcd-client
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following NEW packages will be installed:
  etcd-client
0 upgraded, 1 newly installed, 0 to remove and 1 not upgraded.
Need to get 4,572 kB of archives.
After this operation, 17.2 MB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 etcd-client amd64 3.2.26+dfsg-6ubuntu0.1 [4,572 kB]
Fetched 4,572 kB in 1s (3,705 kB/s)   
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package etcd-client.
(Reading database ... 20430 files and directories currently installed.)
Preparing to unpack .../etcd-client_3.2.26+dfsg-6ubuntu0.1_amd64.deb ...
Unpacking etcd-client (3.2.26+dfsg-6ubuntu0.1) ...
Setting up etcd-client (3.2.26+dfsg-6ubuntu0.1) ...
Processing triggers for man-db (2.9.1-1) ...
```


#### Creating a secret on etcd not emcrypted

```
controlplane ~ ➜  k create secret generic my-secret --from-literal key1=supersecret
secret/my-secret created
```

```
controlplane ~ ➜  k get secret my-secret -o yaml
apiVersion: v1
data:
  key1: c3VwZXJzZWNyZXQ=
kind: Secret
metadata:
  creationTimestamp: "2023-02-11T16:25:07Z"
  name: my-secret
  namespace: default
  resourceVersion: "6493"
  uid: 96df59c7-2faf-4873-a5e7-087142cdb52a
type: Opaque
```

```
controlplane ~ ➜  echo "c3VwZXJzZWNyZXQ=" | base64 -d
supersecret
```


#### Confirming the etcd data is not encrypted 

controlplane ~ ➜  ETCDCTL_API=3 etcdctl \
>    --cacert=/etc/kubernetes/pki/etcd/ca.crt   \
>    --cert=/etc/kubernetes/pki/etcd/server.crt \
>    --key=/etc/kubernetes/pki/etcd/server.key  \
>    get /registry/secrets/default/my-secret | hexdump -C
00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
00000010  73 2f 64 65 66 61 75 6c  74 2f 6d 79 2d 73 65 63  |s/default/my-sec|
00000020  72 65 74 0a 6b 38 73 00  0a 0c 0a 02 76 31 12 06  |ret.k8s.....v1..|
00000030  53 65 63 72 65 74 12 d0  01 0a b0 01 0a 09 6d 79  |Secret........my|
00000040  2d 73 65 63 72 65 74 12  00 1a 07 64 65 66 61 75  |-secret....defau|
00000050  6c 74 22 00 2a 24 39 36  64 66 35 39 63 37 2d 32  |lt".*$96df59c7-2|
00000060  66 61 66 2d 34 38 37 33  2d 61 35 65 37 2d 30 38  |faf-4873-a5e7-08|
00000070  37 31 34 32 63 64 62 35  32 61 32 00 38 00 42 08  |7142cdb52a2.8.B.|
00000080  08 e3 82 9f 9f 06 10 00  8a 01 61 0a 0e 6b 75 62  |..........a..kub|
00000090  65 63 74 6c 2d 63 72 65  61 74 65 12 06 55 70 64  |ectl-create..Upd|
000000a0  61 74 65 1a 02 76 31 22  08 08 e3 82 9f 9f 06 10  |ate..v1"........|
000000b0  00 32 08 46 69 65 6c 64  73 56 31 3a 2d 0a 2b 7b  |.2.FieldsV1:-.+{|
000000c0  22 66 3a 64 61 74 61 22  3a 7b 22 2e 22 3a 7b 7d  |"f:data":{".":{}|
000000d0  2c 22 66 3a 6b 65 79 31  22 3a 7b 7d 7d 2c 22 66  |,"f:key1":{}},"f|
000000e0  3a 74 79 70 65 22 3a 7b  7d 7d 42 00 12 13 0a 04  |:type":{}}B.....|
000000f0  6b 65 79 31 12 0b 73 75  70 65 72 73 65 63 72 65  |key1..supersecre|
00000100  74 1a 06 4f 70 61 71 75  65 1a 00 22 00 0a        |t..Opaque.."..|
0000010e


```
controlplane ~ ✖ ps aux | grep kube-api | grep "encryption-provider-config"
controlplane ~
```

#### Creating Dir and encryption configuration yaml file
```
controlplane ~ ➜  mkdir /etc/kubernetes/enc
```

```
controlplane /etc/kubernetes/enc ➜  head -c 32 /dev/urandom | base64
H0QF0OTY0uz43c3Vif9pf44vaIKVkBQ9OOVSl0T1pVE=
```

```
controlplane ~ ➜  vi enc.yaml
```

```
controlplane /etc/kubernetes/enc ➜  cat enc.yaml 
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: H0QF0OTY0uz43c3Vif9pf44vaIKVkBQ9OOVSl0T1pVE=
      - identity: {}
```




### Adding encryption to kube-apiserver 

```
controlplane ~ ➜  vi /etc/kubernetes/manifests/kube-apiserver.yaml 
...
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=192.92.144.3
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-issuer=https://kubernetes.default.svc.cluster.local
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=10.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    - --encryption-provider-config=/etc/kubernetes/enc/enc.yaml
```

We basically need to add this line to kube-api pod yaml file
```
    - --encryption-provider-config=/etc/kubernetes/enc/enc.yaml
```

And then add:
```
    volumeMounts:
    - name: enc                           # <-- add this line
      mountPath: /etc/kubernetes/enc      # <-- add this line
      readonly: true
...

  volumes:
  - name: enc                             # <-- add this line
    hostPath:                             # <-- add this line
      path: /etc/kubernetes/enc           # <-- add this line
      type: DirectoryOrCreate
```



After changing and saving the file we should confirm that kube-apiserver process is running again and it has encryption enabled

```
controlplane ~ ➜  ps aux | grep kube-api | grep encry
root       17465  0.0  0.1 1055524 301128 ?      Ssl  11:44   0:05 kube-apiserver --advertise-address=192.92.144.3 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key --encryption-provider-config=/etc/kubernetes/enc/enc.yaml
```




#### Creating another secret, this time the etcd has encryption enabled

```
controlplane ~ ➜  kubectl create secret generic my-secret-2 --from-literal key2=topsecret
secret/my-secret-2 created
```

#### Confirming etcd was not enabled for the first secret created previously

```
controlplane ~ ➜  ETCDCTL_API=3 etcdctl    --cacert=/etc/kubernetes/pki/etcd/ca.crt      --cert=/etc/kubernetes/pki/etcd/server.crt    --key=/etc/kubernetes/pki/etcd/server.key     get /registry/secrets/default/my-secret | hexdump -C
00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
00000010  73 2f 64 65 66 61 75 6c  74 2f 6d 79 2d 73 65 63  |s/default/my-sec|
00000020  72 65 74 0a 6b 38 73 00  0a 0c 0a 02 76 31 12 06  |ret.k8s.....v1..|
00000030  53 65 63 72 65 74 12 d0  01 0a b0 01 0a 09 6d 79  |Secret........my|
00000040  2d 73 65 63 72 65 74 12  00 1a 07 64 65 66 61 75  |-secret....defau|
00000050  6c 74 22 00 2a 24 39 36  64 66 35 39 63 37 2d 32  |lt".*$96df59c7-2|
00000060  66 61 66 2d 34 38 37 33  2d 61 35 65 37 2d 30 38  |faf-4873-a5e7-08|
00000070  37 31 34 32 63 64 62 35  32 61 32 00 38 00 42 08  |7142cdb52a2.8.B.|
00000080  08 e3 82 9f 9f 06 10 00  8a 01 61 0a 0e 6b 75 62  |..........a..kub|
00000090  65 63 74 6c 2d 63 72 65  61 74 65 12 06 55 70 64  |ectl-create..Upd|
000000a0  61 74 65 1a 02 76 31 22  08 08 e3 82 9f 9f 06 10  |ate..v1"........|
000000b0  00 32 08 46 69 65 6c 64  73 56 31 3a 2d 0a 2b 7b  |.2.FieldsV1:-.+{|
000000c0  22 66 3a 64 61 74 61 22  3a 7b 22 2e 22 3a 7b 7d  |"f:data":{".":{}|
000000d0  2c 22 66 3a 6b 65 79 31  22 3a 7b 7d 7d 2c 22 66  |,"f:key1":{}},"f|
000000e0  3a 74 79 70 65 22 3a 7b  7d 7d 42 00 12 13 0a 04  |:type":{}}B.....|
000000f0  6b 65 79 31 12 0b 73 75  70 65 72 73 65 63 72 65  |key1..supersecre|
00000100  74 1a 06 4f 70 61 71 75  65 1a 00 22 00 0a        |t..Opaque.."..|
0000010e
```

#### Confirming etcd is encrypted now for the second secret created

```
controlplane ~ ➜  ETCDCTL_API=3 etcdctl \
>    --cacert=/etc/kubernetes/pki/etcd/ca.crt   \
>    --cert=/etc/kubernetes/pki/etcd/server.crt \
>    --key=/etc/kubernetes/pki/etcd/server.key  \
>    get /registry/secrets/default/my-secret-2 | hexdump -C
00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
00000010  73 2f 64 65 66 61 75 6c  74 2f 6d 79 2d 73 65 63  |s/default/my-sec|
00000020  72 65 74 2d 32 0a 6b 38  73 3a 65 6e 63 3a 61 65  |ret-2.k8s:enc:ae|
00000030  73 63 62 63 3a 76 31 3a  6b 65 79 31 3a 33 3b 50  |scbc:v1:key1:3;P|
00000040  c9 3e fb 3c b4 77 f1 74  0c e2 36 40 bd 73 ca 8b  |.>.<.w.t..6@.s..|
00000050  93 e6 e5 39 bf 05 4d 3f  31 a8 76 a3 e5 d7 f1 a2  |...9..M?1.v.....|
00000060  65 d5 25 87 fc e9 98 e5  c9 f8 fb 40 92 fb f2 0a  |e.%........@....|
00000070  d5 8a e8 0d 96 9b 67 78  4d 87 79 0c b8 b7 52 47  |......gxM.y...RG|
00000080  61 1a 72 af d8 ac 28 eb  77 11 df 90 d3 e4 58 a3  |a.r...(.w.....X.|
00000090  58 13 fc 03 62 d3 1d 90  00 c3 e1 6c 2a 2c 7c 77  |X...b......l*,|w|
000000a0  30 05 a6 1c 81 49 1f a6  f7 6a c4 16 4e 24 7f 6d  |0....I...j..N$.m|
000000b0  1d 80 d2 2a 18 dc fd 77  86 11 65 92 78 f4 f2 21  |...*...w..e.x..!|
000000c0  b9 e0 82 d4 e2 69 a5 59  56 ae f4 0c b9 6c 95 4b  |.....i.YV....l.K|
000000d0  0e 43 d1 f3 b5 f5 e3 e7  df 31 c8 98 ca 85 35 61  |.C.......1....5a|
000000e0  5e cd 2e 74 41 7d 48 2a  67 98 90 7f e3 33 9b c0  |^..tA}H*g....3..|
000000f0  13 e0 e5 a7 72 8f d8 1e  26 f8 a8 2a e7 e5 58 bd  |....r...&..*..X.|
00000100  aa 04 83 7d 82 2e cf 21  6d 93 84 95 db 30 2b 86  |...}...!m....0+.|
00000110  5e 6d 71 8a c6 f4 54 46  15 24 ce 4f 5c 80 97 2b  |^mq...TF.$.O\..+|
00000120  97 df 2c 64 12 67 67 3b  67 a3 50 0a fb af 5d 3c  |..,d.gg;g.P...]<|
00000130  eb 8e 43 fe dc 2f d4 f4  a0 46 4c 90 4a 0a        |..C../...FL.J.|
0000013e
```






